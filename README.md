import UIKit

// MARK: - 1. Extensions (Helpers visuales del Código 2)

extension UIColor {
    convenience init(hex: String) {
        var hexSanitized = hex.trimmingCharacters(in: .whitespacesAndNewlines)
        hexSanitized = hexSanitized.replacingOccurrences(of: "#", with: "")
        var rgb: UInt64 = 0
        Scanner(string: hexSanitized).scanHexInt64(&rgb)
        let red = CGFloat((rgb & 0xFF0000) >> 16) / 255.0
        let green = CGFloat((rgb & 0x00FF00) >> 8) / 255.0
        let blue = CGFloat(rgb & 0x0000FF) / 255.0
        self.init(red: red, green: green, blue: blue, alpha: 1.0)
    }
}

extension UIImage {
    static func gradientImage(bounds: CGRect, colors: [UIColor], startPoint: CGPoint, endPoint: CGPoint) -> UIImage {
        let gradientLayer = CAGradientLayer()
        gradientLayer.frame = bounds
        gradientLayer.colors = colors.map { $0.cgColor }
        gradientLayer.startPoint = startPoint
        gradientLayer.endPoint = endPoint
        
        let renderer = UIGraphicsImageRenderer(bounds: bounds)
        return renderer.image { ctx in
            gradientLayer.render(in: ctx.cgContext)
        }
    }
}

// MARK: - 2. Custom Navigation Controller (El Cerebro)



class NavigationInterceptor: NSObject, UINavigationControllerDelegate {
    
    weak var originalDelegate: UINavigationControllerDelegate?
    var myWillShowCallback: ((UIViewController) -> Void)?
    
    init(originalDelegate: UINavigationControllerDelegate?) {
        self.originalDelegate = originalDelegate
        super.init()
    }
    
    // MARK: - La Intercepción (Aquí ocurre la magia)
    
    func navigationController(_ navigationController: UINavigationController, willShow viewController: UIViewController, animated: Bool) {
        
        // PASO 1: "super.willShow"
        // Llamamos al delegado original primero.
        originalDelegate?.navigationController?(navigationController, willShow: viewController, animated: animated)
        
        // PASO 2: Tu lógica
        // Esto se ejecuta DESPUÉS de que el nav controller haya hecho su configuración.
        myWillShowCallback?(viewController)
    }
    
    // MARK: - Forwarding (Transparencia)
    
    // MARK: - Forwarding (Corrección de Concurrencia)

    // Agregamos 'nonisolated' para coincidir con la definición base de NSObject
    override nonisolated func forwardingTarget(for aSelector: Selector!) -> Any? {
        // Usamos assumeIsolated porque sabemos que UIKit llama a esto desde el hilo principal
        // y necesitamos acceder a 'originalDelegate' que pertenece al MainActor.
        return MainActor.assumeIsolated {
            return originalDelegate
        }
    }

    override nonisolated func responds(to aSelector: Selector!) -> Bool {
        if super.responds(to: aSelector) { return true }
        
        return MainActor.assumeIsolated {
            return originalDelegate?.responds(to: aSelector) ?? false
        }
    }
}

class NavigationController: UINavigationController, UINavigationControllerDelegate {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.delegate = self
        // Configuración base
        navigationBar.prefersLargeTitles = true
    }
    
    // Esta función centraliza TODA la lógica de apariencia
    func navigationController(_ navigationController: UINavigationController, willShow viewController: UIViewController, animated: Bool) {
        
        // 1. Siempre permitir Large Titles por defecto
        viewController.navigationItem.largeTitleDisplayMode = .always
        
        // 2. Preparar variables
        let appearance = UINavigationBarAppearance()
        let scrollEdgeAppearance = UINavigationBarAppearance()
        let tintColor: UIColor
        let isTranslucent: Bool
        
        // 3. Lógica condicional según el VC
        if viewController is ViewControllerTwo {
            // --- CONFIGURACIÓN GRADIENTE (Código 2 adaptado) ---
            
            let colorTop = UIColor(hex: "#001D46")
            let colorBottom = UIColor(hex: "#004692")
            // Usamos el ancho de la vista del navigation controller
            let width = navigationController.view.frame.width
            
            // Generar imagen (altura suficiente para cubrir status bar + large title)
            let gradientImage = UIImage.gradientImage(
                bounds: CGRect(x: 0, y: 0, width: width, height: 200),
                colors: [colorTop, colorBottom],
                startPoint: CGPoint(x: 0, y: 0),
                endPoint: CGPoint(x: 1, y: 1)
            )
            
            let textAttributes: [NSAttributedString.Key: Any] = [.foregroundColor: UIColor.white]
            
            // Standard (cuando scrolleas hacia abajo y la barra se contrae)
            appearance.configureWithOpaqueBackground()
            appearance.backgroundImage = gradientImage
            appearance.titleTextAttributes = textAttributes
            appearance.largeTitleTextAttributes = textAttributes
            
            // ScrollEdge (cuando estás arriba del todo - Large Title)
            // IMPORTANTE: Usamos configureWithOpaqueBackground con la imagen para que sea sólido
            scrollEdgeAppearance.configureWithOpaqueBackground()
            scrollEdgeAppearance.backgroundImage = gradientImage
            scrollEdgeAppearance.titleTextAttributes = textAttributes
            scrollEdgeAppearance.largeTitleTextAttributes = textAttributes
            
            tintColor = .white
            // TRUCO CLAVE: Al poner false, el sistema ajusta el safeArea para que el contenido empiece DEBAJO de la barra sólida
            isTranslucent = false
            
        } else {
            // --- CONFIGURACIÓN ESTÁNDAR BLANCA (VC1, VC3) ---
            
            // Standard (Barra compacta)
            appearance.configureWithOpaqueBackground()
            appearance.backgroundColor = .systemBackground
            appearance.shadowColor = nil // Opcional: quitar la línea separadora
            
            // ScrollEdge (Large Title)
            // Usamos Transparente aquí para el efecto clásico de iOS donde el fondo es el view del controlador
            scrollEdgeAppearance.configureWithTransparentBackground()
            
            tintColor = .systemBlue
            // TRUCO CLAVE: Al poner true, permitimos que el Large Title se vea limpio sobre el fondo blanco del VC
            isTranslucent = true
        }
        
        // 4. Aplicar configuraciones al NavigationBar global
        navigationBar.standardAppearance = appearance
        navigationBar.compactAppearance = appearance
        navigationBar.scrollEdgeAppearance = scrollEdgeAppearance
        
        // 5. Animación suave de las propiedades
        if let coordinator = viewController.transitionCoordinator {
            coordinator.animate { _ in
                self.navigationBar.tintColor = tintColor
                self.navigationBar.isTranslucent = isTranslucent
                self.navigationBar.layoutIfNeeded()
            }
        } else {
            navigationBar.tintColor = tintColor
            navigationBar.isTranslucent = isTranslucent
        }
    }
}

// MARK: - 3. Layout (Sticky Carousel)

@MainActor
final class StickyCarouselHeaderLayout: UICollectionViewCompositionalLayout {
    var stickyHeaderSection: Int = 1
    let stickyHeaderZIndex: Int = 1000
    let standardHeaderZIndex: Int = 100
    
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        guard let superAttributes = super.layoutAttributesForElements(in: rect),
              let collectionView else { return nil }
        
        var mutableAttributes = superAttributes.compactMap { $0.copy() as? UICollectionViewLayoutAttributes }
        let contentInsetTop = collectionView.adjustedContentInset.top
        let effectiveOffsetY = collectionView.contentOffset.y + contentInsetTop
        let stickyHeaderIndexPath = IndexPath(item: 0, section: stickyHeaderSection)
        
        var stickyHeaderAttrs: UICollectionViewLayoutAttributes?
        var needsToAddStickyHeader = false
        
        for attributes in mutableAttributes where attributes.representedElementKind == UICollectionView.elementKindSectionHeader {
            if attributes.indexPath == stickyHeaderIndexPath {
                stickyHeaderAttrs = attributes
                let originalStickyHeaderY = super.layoutAttributesForSupplementaryView(ofKind: UICollectionView.elementKindSectionHeader, at: stickyHeaderIndexPath)?.frame.origin.y ?? attributes.frame.origin.y
                let stickyY = max(effectiveOffsetY, originalStickyHeaderY)
                attributes.frame.origin.y = stickyY
                attributes.zIndex = stickyHeaderZIndex
            } else {
                attributes.zIndex = standardHeaderZIndex
            }
        }
        
        if stickyHeaderAttrs == nil {
            if let fetchedStickyAttrs = super.layoutAttributesForSupplementaryView(ofKind: UICollectionView.elementKindSectionHeader, at: stickyHeaderIndexPath)?.copy() as? UICollectionViewLayoutAttributes {
                let originalStickyHeaderY = super.layoutAttributesForSupplementaryView(ofKind: UICollectionView.elementKindSectionHeader, at: stickyHeaderIndexPath)?.frame.origin.y ?? fetchedStickyAttrs.frame.origin.y
                let stickyY = max(effectiveOffsetY, originalStickyHeaderY)
                fetchedStickyAttrs.frame.origin.y = stickyY
                fetchedStickyAttrs.zIndex = stickyHeaderZIndex
                if fetchedStickyAttrs.frame.intersects(rect) {
                    stickyHeaderAttrs = fetchedStickyAttrs
                    needsToAddStickyHeader = true
                }
            }
        }
        
        if needsToAddStickyHeader, let attrsToAdd = stickyHeaderAttrs {
            if !mutableAttributes.contains(where: { $0.indexPath == attrsToAdd.indexPath && $0.representedElementKind == attrsToAdd.representedElementKind }) {
                mutableAttributes.append(attrsToAdd)
            }
        }
        return mutableAttributes
    }
    
    override func shouldInvalidateLayout(forBoundsChange newBounds: CGRect) -> Bool {
        return true
    }
    
    override func layoutAttributesForSupplementaryView(ofKind elementKind: String, at indexPath: IndexPath) -> UICollectionViewLayoutAttributes? {
        guard let attributes = super.layoutAttributesForSupplementaryView(ofKind: elementKind, at: indexPath)?.copy() as? UICollectionViewLayoutAttributes else { return nil }
        if elementKind == UICollectionView.elementKindSectionHeader {
            if indexPath.section == stickyHeaderSection {
                guard let collectionView else { return attributes }
                let contentInsetTop = collectionView.adjustedContentInset.top
                let effectiveOffsetY = collectionView.contentOffset.y + contentInsetTop
                let originalStickyHeaderY = super.layoutAttributesForSupplementaryView(ofKind: elementKind, at: indexPath)?.frame.origin.y ?? attributes.frame.origin.y
                attributes.frame.origin.y = max(effectiveOffsetY, originalStickyHeaderY)
                attributes.zIndex = stickyHeaderZIndex
            } else {
                attributes.zIndex = standardHeaderZIndex
            }
        }
        return attributes
    }
}

// MARK: - 4. Views & Cells

class SectionHeaderView: UICollectionReusableView {
    static let reuseId = "SectionHeaderView"
    let titleLabel = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        configure()
    }
    
    required init?(coder: NSCoder) { fatalError() }
    
    private func configure() {
        backgroundColor = .systemBackground // Importante para cubrir el contenido al hacer sticky
        addSubview(titleLabel)
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        titleLabel.font = UIFont.systemFont(ofSize: 22, weight: .bold)
        NSLayoutConstraint.activate([
            titleLabel.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 5),
            titleLabel.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -5),
            titleLabel.topAnchor.constraint(equalTo: topAnchor),
            titleLabel.bottomAnchor.constraint(equalTo: bottomAnchor)
        ])
    }
}

class TopInfoCell: UICollectionViewCell {
    static let reuseId = "TopInfoCell"
    let label = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupUI()
    }
    
    required init?(coder: NSCoder) { fatalError() }
    
    private func setupUI() {
        contentView.backgroundColor = .systemGray6
        contentView.layer.cornerRadius = 8
        label.font = .systemFont(ofSize: 18, weight: .medium)
        label.textColor = .label
        contentView.addSubview(label)
        label.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            label.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            label.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),
            label.centerYAnchor.constraint(equalTo: contentView.centerYAnchor)
        ])
    }
    
    func configure(text: String) {
        label.text = text
    }
}

class GridItemCell: UICollectionViewCell {
    static let reuseId = "GridItemCell"
    let label = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupUI()
    }
    
    required init?(coder: NSCoder) { fatalError() }
    
    private func setupUI() {
        label.font = .systemFont(ofSize: 17)
        label.textColor = .label
        label.textAlignment = .center
        contentView.addSubview(label)
        label.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: contentView.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: contentView.centerYAnchor)
        ])
    }
    
    func configure(text: String, backgroundColor: UIColor) {
        label.text = text
        contentView.backgroundColor = backgroundColor
    }
}

// MARK: - 5. View Controllers

// Base Controller para compartir lógica de layout (opcional, reduce código duplicado)
class BaseGridViewController: UIViewController, UICollectionViewDataSource, UICollectionViewDelegate {
    var collectionView: UICollectionView!
    var gridData = Array(1...30)
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        configureCollectionView()
    }
    
    func configureCollectionView() {
        let layout = StickyCarouselHeaderLayout { [weak self] i, _ in
            return self?.createSectionLayout(for: i) ?? NSCollectionLayoutSection(group: NSCollectionLayoutGroup(layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1), heightDimension: .absolute(1))))
        }
        layout.stickyHeaderSection = 1
        
        collectionView = UICollectionView(frame: .zero, collectionViewLayout: layout)
        collectionView.backgroundColor = .systemBackground
        
        collectionView.register(TopInfoCell.self, forCellWithReuseIdentifier: TopInfoCell.reuseId)
        collectionView.register(GridItemCell.self, forCellWithReuseIdentifier: GridItemCell.reuseId)
        collectionView.register(SectionHeaderView.self, forSupplementaryViewOfKind: UICollectionView.elementKindSectionHeader, withReuseIdentifier: SectionHeaderView.reuseId)
        
        collectionView.dataSource = self
        collectionView.delegate = self
        
        view.addSubview(collectionView)
        collectionView.translatesAutoresizingMaskIntoConstraints = false
        
        NSLayoutConstraint.activate([
            collectionView.topAnchor.constraint(equalTo: view.topAnchor),
            collectionView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            collectionView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            collectionView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }
    
    func createSectionLayout(for sectionIndex: Int) -> NSCollectionLayoutSection {
        if sectionIndex == 0 {
            let item = NSCollectionLayoutItem(layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0), heightDimension: .fractionalHeight(1.0)))
            item.contentInsets = NSDirectionalEdgeInsets(top: 2, leading: 10, bottom: 2, trailing: 10)
            let group = NSCollectionLayoutGroup.horizontal(layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0), heightDimension: .absolute(60)), subitems: [item])
            let section = NSCollectionLayoutSection(group: group)
            section.contentInsets = NSDirectionalEdgeInsets(top: 10, leading: 0, bottom: 20, trailing: 0)
            return section
        } else {
            let item = NSCollectionLayoutItem(layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.5), heightDimension: .absolute(100)))
            item.contentInsets = NSDirectionalEdgeInsets(top: 5, leading: 5, bottom: 5, trailing: 5)
            let group = NSCollectionLayoutGroup.horizontal(layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0), heightDimension: .absolute(100)), subitems: [item])
            let section = NSCollectionLayoutSection(group: group)
            section.boundarySupplementaryItems = [NSCollectionLayoutBoundarySupplementaryItem(layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0), heightDimension: .absolute(44)), elementKind: UICollectionView.elementKindSectionHeader, alignment: .top)]
            return section
        }
    }
    
    // Default implementations
    func numberOfSections(in collectionView: UICollectionView) -> Int { return 2 }
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int { return section == 0 ? 2 : gridData.count }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        return UICollectionViewCell() // Override in subclasses
    }
    
    func collectionView(_ collectionView: UICollectionView, viewForSupplementaryElementOfKind kind: String, at indexPath: IndexPath) -> UICollectionReusableView {
        let h = collectionView.dequeueReusableSupplementaryView(ofKind: kind, withReuseIdentifier: SectionHeaderView.reuseId, for: indexPath) as! SectionHeaderView
        h.titleLabel.text = "Grid"
        h.titleLabel.textColor = .systemGreen
        return h
    }
    
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {}
}

// --- CLASES ESPECÍFICAS ---

class ViewControllerOne: BaseGridViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "One"
        extendedLayoutIncludesOpaqueBars = true
    }
    
    override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        if indexPath.section == 0 {
            let c = collectionView.dequeueReusableCell(withReuseIdentifier: TopInfoCell.reuseId, for: indexPath) as! TopInfoCell
            c.configure(text: "Top Cell \(indexPath.row + 1)")
            return c
        } else {
            let c = collectionView.dequeueReusableCell(withReuseIdentifier: GridItemCell.reuseId, for: indexPath) as! GridItemCell
            c.configure(text: "Item \(gridData[indexPath.row])", backgroundColor: .systemOrange)
            return c
        }
    }
    
    override func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        if indexPath.section == 1 {
            navigationController?.pushViewController(ViewControllerTwo(), animated: true)
        }
    }
}

class ViewControllerTwo: BaseGridViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Two"
    }
    
    override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        if indexPath.section == 0 {
            let c = collectionView.dequeueReusableCell(withReuseIdentifier: TopInfoCell.reuseId, for: indexPath) as! TopInfoCell
            c.configure(text: "Top Cell \(indexPath.row + 1)")
            return c
        } else {
            let c = collectionView.dequeueReusableCell(withReuseIdentifier: GridItemCell.reuseId, for: indexPath) as! GridItemCell
            c.configure(text: "Item \(gridData[indexPath.row])", backgroundColor: .systemGreen)
            return c
        }
    }
    
    override func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        if indexPath.section == 1 {
            navigationController?.pushViewController(ViewControllerThree(), animated: true)
        }
    }
    
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        // Llamada a super si fuera necesario (aunque en el base no hace nada)
        // super.scrollViewDidScroll(scrollView)
        
        // Tu lógica antigua
        let offsetY = scrollView.contentOffset.y
        let contentInsetTop = scrollView.contentInset.top
        let threshold: CGFloat = -contentInsetTop + 50
        
        if offsetY < threshold {
            // Estamos arriba (Large Title)
            if navigationController?.navigationBar.isTranslucent == true {
                 navigationController?.navigationBar.isTranslucent = false
            }
        } else {
            // Estamos scrolleando (Inline Title)
            if navigationController?.navigationBar.isTranslucent == false {
                 navigationController?.navigationBar.isTranslucent = true
            }
        }
    }
}

class ViewControllerThree: BaseGridViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Three"
        extendedLayoutIncludesOpaqueBars = true
    }
    
    override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        if indexPath.section == 0 {
            let c = collectionView.dequeueReusableCell(withReuseIdentifier: TopInfoCell.reuseId, for: indexPath) as! TopInfoCell
            c.configure(text: "Top Cell \(indexPath.row + 1)")
            return c
        } else {
            let c = collectionView.dequeueReusableCell(withReuseIdentifier: GridItemCell.reuseId, for: indexPath) as! GridItemCell
            c.configure(text: "Item \(gridData[indexPath.row])", backgroundColor: .systemBlue)
            return c
        }
    }
}
