import UIKit

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

class NavigationInterceptor: NSObject, UINavigationControllerDelegate {
    
    weak var originalDelegate: UINavigationControllerDelegate?
    var myWillShowCallback: ((UIViewController) -> Void)?
    
    init(originalDelegate: UINavigationControllerDelegate?) {
        self.originalDelegate = originalDelegate
        super.init()
    }
    
    func navigationController(_ navigationController: UINavigationController, willShow viewController: UIViewController, animated: Bool) {
        originalDelegate?.navigationController?(navigationController, willShow: viewController, animated: animated)
        myWillShowCallback?(viewController)
    }
    
    override nonisolated func forwardingTarget(for aSelector: Selector!) -> Any? {
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
        navigationBar.prefersLargeTitles = true
    }
    
    func navigationController(_ navigationController: UINavigationController, willShow viewController: UIViewController, animated: Bool) {
        
        viewController.navigationItem.largeTitleDisplayMode = .always
        
        let appearance = UINavigationBarAppearance()
        let scrollEdgeAppearance = UINavigationBarAppearance()
        let tintColor: UIColor
        
        if viewController is ViewControllerTwo {
            
            let colorTop = UIColor(hex: "#001D46")
            let colorBottom = UIColor(hex: "#004692")
            let textAttributes: [NSAttributedString.Key: Any] = [.foregroundColor: UIColor.white]
            
            let width = navigationController.view.bounds.width
            
            let gradientImage = UIImage.gradientImage(
                bounds: CGRect(x: 0, y: 0, width: width, height: 100),
                colors: [colorTop, colorBottom],
                startPoint: CGPoint(x: 0, y: 0),
                endPoint: CGPoint(x: 1, y: 1)
            )
            
            appearance.configureWithOpaqueBackground()
            appearance.backgroundImage = gradientImage
            appearance.titleTextAttributes = textAttributes
            appearance.shadowColor = nil
            
            scrollEdgeAppearance.configureWithTransparentBackground()
            scrollEdgeAppearance.largeTitleTextAttributes = textAttributes
            
            tintColor = .white
            navigationBar.isTranslucent = true
            
        } else {
            appearance.configureWithOpaqueBackground()
            appearance.backgroundColor = .systemBackground
            appearance.shadowColor = nil
            
            scrollEdgeAppearance.configureWithTransparentBackground()
            
            tintColor = .systemBlue
            navigationBar.isTranslucent = true
        }
        
        navigationBar.standardAppearance = appearance
        navigationBar.compactAppearance = appearance
        navigationBar.scrollEdgeAppearance = scrollEdgeAppearance
        
        if let coordinator = viewController.transitionCoordinator {
            coordinator.animate { _ in
                self.navigationBar.tintColor = tintColor
                self.navigationBar.layoutIfNeeded()
            }
        } else {
            navigationBar.tintColor = tintColor
        }
    }
}

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

class SectionHeaderView: UICollectionReusableView {
    static let reuseId = "SectionHeaderView"
    let titleLabel = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        configure()
    }
    
    required init?(coder: NSCoder) { fatalError() }
    
    private func configure() {
        backgroundColor = .systemBackground
        
        addSubview(titleLabel)
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        titleLabel.font = UIFont.systemFont(ofSize: 22, weight: .bold)
        NSLayoutConstraint.activate([
            titleLabel.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 16),
            titleLabel.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -16),
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
        backgroundColor = .clear
        contentView.backgroundColor = .clear
        
        label.font = .preferredFont(forTextStyle: .body)
        label.adjustsFontForContentSizeCategory = true
        label.numberOfLines = 0
        
        label.textColor = .white
        label.backgroundColor = .systemGray
        label.layer.cornerRadius = 6
        label.layer.borderWidth = 1
        label.layer.borderColor = UIColor.systemGray6.cgColor
        
        contentView.addSubview(label)
        label.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            label.heightAnchor.constraint(greaterThanOrEqualToConstant: 30),
            label.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            label.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),
            label.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 16),
            label.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -16)
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
        label.textColor = .white
        label.textAlignment = .center
        contentView.addSubview(label)
        label.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: contentView.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: contentView.centerYAnchor)
        ])
        contentView.layer.cornerRadius = 8
    }
    
    func configure(text: String, backgroundColor: UIColor) {
        label.text = text
        contentView.backgroundColor = backgroundColor
    }
}

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
            let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                                  heightDimension: .estimated(60))
            let item = NSCollectionLayoutItem(layoutSize: itemSize)
            
            let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                                   heightDimension: .estimated(60))
            
            let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize,
                                                           subitems: [item])
            
            let section = NSCollectionLayoutSection(group: group)
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
    
    func numberOfSections(in collectionView: UICollectionView) -> Int {
        return 2
    }
    
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return section == 0 ? 2 : gridData.count
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        return UICollectionViewCell()
    }
    
    func collectionView(_ collectionView: UICollectionView, viewForSupplementaryElementOfKind kind: String, at indexPath: IndexPath) -> UICollectionReusableView {
        let h = collectionView.dequeueReusableSupplementaryView(ofKind: kind, withReuseIdentifier: SectionHeaderView.reuseId, for: indexPath) as! SectionHeaderView
        h.titleLabel.text = "Grid Section"
        h.titleLabel.textColor = .systemGreen
        return h
    }
    
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {}
}

class ViewControllerOne: BaseGridViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "One"
    }
    
    override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        if indexPath.section == 0 {
            let c = collectionView.dequeueReusableCell(withReuseIdentifier: GridItemCell.reuseId, for: indexPath) as! GridItemCell
            c.configure(text: "Top \(indexPath.row + 1)", backgroundColor: .systemGray6)
            c.label.textColor = .black
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
    
    private var gradientContainerView: UIView?
    private var gradientLayer: CAGradientLayer?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Two"
        view.backgroundColor = .systemBackground
        collectionView.backgroundColor = .clear
        setupScrollableGradient()
    }
    
    override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
        super.traitCollectionDidChange(previousTraitCollection)
        
        if previousTraitCollection?.preferredContentSizeCategory != traitCollection.preferredContentSizeCategory {
            collectionView.collectionViewLayout.invalidateLayout()
            collectionView.layoutIfNeeded()
            updateGradientFrame()
        }
    }
    
    private func setupScrollableGradient() {
        let container = UIView()
        collectionView.insertSubview(container, at: 0)
        self.gradientContainerView = container
        
        container.layer.zPosition = -1
        
        let layer = CAGradientLayer()
        layer.colors = [
            UIColor(hex: "#001D46").cgColor,
            UIColor(hex: "#004692").cgColor
        ]
        layer.startPoint = CGPoint(x: 0, y: 0)
        layer.endPoint = CGPoint(x: 1, y: 1)
        container.layer.addSublayer(layer)
        self.gradientLayer = layer
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        updateGradientFrame()
    }
    
    private func updateGradientFrame() {
        let extraTopSpace: CGFloat = 3000
        let targetIndexPath = IndexPath(item: 1, section: 0)
        var targetFrame: CGRect = .zero
        
        if let cell = collectionView.cellForItem(at: targetIndexPath) {
            targetFrame = cell.frame
        } else if let attributes = collectionView.layoutAttributesForItem(at: targetIndexPath) {
            targetFrame = attributes.frame
        }
        
        guard targetFrame != .zero, let container = gradientContainerView else {
            return
        }
        
        let cutOffY = targetFrame.midY
        let totalHeight = extraTopSpace + cutOffY
        
        if container.frame.height != totalHeight || container.frame.width != collectionView.bounds.width {
            container.frame = CGRect(x: 0, y: -extraTopSpace, width: collectionView.bounds.width, height: totalHeight)
            gradientLayer?.frame = container.bounds
        }
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
}

class ViewControllerThree: BaseGridViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Three"
    }
    
    override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        if indexPath.section == 0 {
            let c = collectionView.dequeueReusableCell(withReuseIdentifier: GridItemCell.reuseId, for: indexPath) as! GridItemCell
            c.configure(text: "Top \(indexPath.row + 1)", backgroundColor: .systemGray6)
            c.label.textColor = .black
            return c
        } else {
            let c = collectionView.dequeueReusableCell(withReuseIdentifier: GridItemCell.reuseId, for: indexPath) as! GridItemCell
            c.configure(text: "Item \(gridData[indexPath.row])", backgroundColor: .systemBlue)
            return c
        }
    }
}
