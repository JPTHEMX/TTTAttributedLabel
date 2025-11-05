import UIKit

class CustomButtonControl: UIControl {
    
    enum ButtonType {
        case dualLabel
        case imageWithLabel
    }
    
    private let containerStackView: UIStackView = {
        let stack = UIStackView()
        stack.axis = .vertical
        stack.spacing = 2
        stack.alignment = .fill
        stack.distribution = .fill
        stack.isUserInteractionEnabled = false
        stack.translatesAutoresizingMaskIntoConstraints = false
        stack.isLayoutMarginsRelativeArrangement = true
        stack.layoutMargins = UIEdgeInsets(top: 8, left: 8, bottom: 8, right: 8)
        return stack
    }()
    
    private let topLabel: UILabel = {
        let label = UILabel()
        label.font = UIFont.preferredFont(forTextStyle: .caption1)
        label.textAlignment = .center
        label.adjustsFontForContentSizeCategory = true
        label.numberOfLines = 0
        label.lineBreakMode = .byWordWrapping
        label.setContentCompressionResistancePriority(.defaultHigh, for: .horizontal)
        label.setContentCompressionResistancePriority(.required, for: .vertical)
        return label
    }()
    
    private let bottomLabel: UILabel = {
        let label = UILabel()
        label.font = UIFont.preferredFont(forTextStyle: .caption2)
        label.textAlignment = .center
        label.adjustsFontForContentSizeCategory = true
        label.numberOfLines = 0
        label.lineBreakMode = .byWordWrapping
        label.setContentCompressionResistancePriority(.defaultHigh, for: .horizontal)
        label.setContentCompressionResistancePriority(.required, for: .vertical)
        return label
    }()
    
    private let imageView: UIImageView = {
        let imageView = UIImageView()
        imageView.contentMode = .scaleAspectFill
        imageView.tintColor = .label
        imageView.setContentCompressionResistancePriority(.required, for: .horizontal)
        imageView.setContentCompressionResistancePriority(.required, for: .vertical)
        imageView.translatesAutoresizingMaskIntoConstraints = false
        return imageView
    }()
    
    private let buttonType: ButtonType
    
    var topText: String? {
        didSet {
            topLabel.text = topText
        }
    }
    
    var bottomText: String? {
        didSet {
            updateBottomLabelWithUnderline()
        }
    }
    
    var image: UIImage? {
        didSet {
            imageView.image = image
        }
    }
    
    init(type: ButtonType) {
        self.buttonType = type
        super.init(frame: .zero)
        setupView()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setupView() {
        addSubview(containerStackView)
        
        switch buttonType {
        case .dualLabel:
            containerStackView.addArrangedSubview(topLabel)
            containerStackView.addArrangedSubview(bottomLabel)
            
            NSLayoutConstraint.activate([
                topLabel.heightAnchor.constraint(greaterThanOrEqualToConstant: 24)
            ])
            
        case .imageWithLabel:
            let imageContainer = UIView()
            imageContainer.addSubview(imageView)
            containerStackView.addArrangedSubview(imageContainer)
            containerStackView.addArrangedSubview(bottomLabel)
            
            NSLayoutConstraint.activate([
                imageContainer.heightAnchor.constraint(equalToConstant: 24),
                imageView.widthAnchor.constraint(equalToConstant: 24),
                imageView.heightAnchor.constraint(equalToConstant: 24),
                imageView.centerXAnchor.constraint(equalTo: imageContainer.centerXAnchor),
                imageView.centerYAnchor.constraint(equalTo: imageContainer.centerYAnchor),
                imageView.topAnchor.constraint(equalTo: imageContainer.topAnchor),
                imageView.bottomAnchor.constraint(equalTo: imageContainer.bottomAnchor)
            ])
        }
        
        NSLayoutConstraint.activate([
            containerStackView.topAnchor.constraint(equalTo: topAnchor),
            containerStackView.leadingAnchor.constraint(equalTo: leadingAnchor),
            containerStackView.trailingAnchor.constraint(equalTo: trailingAnchor),
            containerStackView.bottomAnchor.constraint(equalTo: bottomAnchor)
        ])
        
        
        isAccessibilityElement = true
        accessibilityTraits = .button
    }
    
    private func updateBottomLabelWithUnderline() {
        guard let text = bottomText else {
            bottomLabel.attributedText = nil
            return
        }
        
        // --- CAMBIO AQUÍ: Corrección para respetar la fuente existente ---
        let existingFont = bottomLabel.font ?? UIFont.preferredFont(forTextStyle: .caption2)
        
        let attributes: [NSAttributedString.Key: Any] = [
            .font: existingFont,
            .underlineStyle: NSUnderlineStyle.single.rawValue,
            .underlineColor: UIColor.label,
        ]
        
        let attributedString = NSAttributedString(string: text, attributes: attributes)
        bottomLabel.attributedText = attributedString
        
        if buttonType == .dualLabel {
            accessibilityLabel = "\(topText ?? ""), \(bottomText ?? "")"
        } else {
            accessibilityLabel = bottomText
        }
    }
    
    override var isHighlighted: Bool {
        didSet {
            alpha = isHighlighted ? 0.7 : 1.0
        }
    }
    
    override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
        super.traitCollectionDidChange(previousTraitCollection)
        
        if traitCollection.userInterfaceStyle != previousTraitCollection?.userInterfaceStyle {
            updateBottomLabelWithUnderline()
        }
    }
}








import UIKit

class EmptyCell: UICollectionViewCell {
    
    static let reuseIdentifier = "EmptyCell"
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        contentView.backgroundColor = .systemGray6
        contentView.layer.cornerRadius = 8
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}








import UIKit

enum ExperienceType {
    case `default`
    case shopping
}

class ContentCell: UICollectionViewCell {
    
    static let reuseIdentifier = "ContentCell"
    
    // MARK: - UI Components
    
    lazy var titleLabel: UILabel = {
            let label = UILabel()
            label.text = "Title"
            
            let baseFont = UIFont.systemFont(ofSize: 24, weight: .bold)
            label.font = UIFontMetrics(forTextStyle: .title1).scaledFont(for: baseFont)
            label.adjustsFontForContentSizeCategory = true
            label.numberOfLines = 1
            label.lineBreakMode = .byWordWrapping
            label.setContentCompressionResistancePriority(.required, for: .horizontal)
            label.setContentCompressionResistancePriority(.defaultLow, for: .vertical)
            label.setContentHuggingPriority(.defaultLow, for: .horizontal)
            
            return label
        }()
    
    lazy var subTitleLabel: UILabel = {
            let label = UILabel()
            label.text = "Sub title"
            
            let baseFont = UIFont.systemFont(ofSize: 24, weight: .bold)
            label.font = UIFontMetrics(forTextStyle: .title1).scaledFont(for: baseFont)
            label.adjustsFontForContentSizeCategory = true
            label.numberOfLines = 1
            label.lineBreakMode = .byWordWrapping
            label.setContentCompressionResistancePriority(.required, for: .horizontal)
            label.setContentCompressionResistancePriority(.defaultLow, for: .vertical)
            label.setContentHuggingPriority(.defaultLow, for: .horizontal)
            
            return label
        }()
    
    lazy var labelsStackView: UIStackView = {
        let stack = UIStackView(arrangedSubviews: [titleLabel, subTitleLabel])
        stack.axis = .vertical
        stack.distribution = .fill
        stack.spacing = 8
        stack.alignment = .fill
        return stack
    }()

    lazy var buttonStackView: UIStackView = {
        let stack = UIStackView()
        stack.axis = .horizontal
        stack.distribution = .fill
        stack.spacing = 8
        stack.alignment = .fill
        stack.setContentCompressionResistancePriority(.defaultHigh - 1, for: .horizontal)
        return stack
    }()

    lazy var mainStackView: UIStackView = {
        let stack = UIStackView(arrangedSubviews: [labelsStackView, buttonStackView])
        stack.axis = .horizontal
        stack.distribution = .fill
        stack.alignment = .center
        stack.spacing = 16
        stack.translatesAutoresizingMaskIntoConstraints = false
        return stack
    }()
    
    // --- Vistas condicionales (Cambiadas a lazy var) ---
    lazy var firstSeparator: UIView = self.createSeparator()
    
    private lazy var secondSeparator: UIView = self.createSeparator()
    
    private lazy var button1: CustomButtonControl = {
        let button = self.createDualLabelButton(topText: "100", bottomText: "Title Label One")
        button.addTarget(self, action: #selector(self.buttonTapped(_:)), for: .touchUpInside)
        return button
    }()
    
    private lazy var button2: CustomButtonControl = {
        let button = self.createDualLabelButton(topText: "200", bottomText: "Title Label Two")
        button.addTarget(self, action: #selector(self.buttonTapped(_:)), for: .touchUpInside)
        return button
    }()
    
    private lazy var button3: CustomButtonControl = {
        let button = self.createImageButton(image: UIImage(systemName: "photo"), bottomText: "Title Label Three")
        button.addTarget(self, action: #selector(self.buttonTapped(_:)), for: .touchUpInside)
        return button
    }()
    
    private lazy var button4: CustomButtonControl = {
        let button = self.createImageButton(image: UIImage(systemName: "star"), bottomText: "Button Four")
        button.addTarget(self, action: #selector(self.buttonTapped(_:)), for: .touchUpInside)
        return button
    }()
    
    private lazy var button5: CustomButtonControl = {
        let button = self.createImageButton(image: UIImage(systemName: "heart"), bottomText: "Button Five")
        button.addTarget(self, action: #selector(self.buttonTapped(_:)), for: .touchUpInside)
        return button
    }()
    
    private lazy var spacerView: UIView = UIView()
    
    private lazy var button1WidthConstraint: NSLayoutConstraint = {
        return self.button1.widthAnchor.constraint(equalTo: self.button2.widthAnchor)
    }()
    
    private lazy var buttonWidthConstraint: NSLayoutConstraint = {
        return self.button2.widthAnchor.constraint(equalTo: self.button3.widthAnchor)
    }()
    
    private var currentType: ExperienceType = .shopping // Estado para layout

    // MARK: - Initialization
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupLayout()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    // MARK: - Setup (Refactorizado para usar lazy vars)
    
    func setupLayout() {
        
        // --- Añadir todas las vistas al stack en orden ---
        // Al acceder a ellas, se inicializan las lazy vars
        buttonStackView.addArrangedSubview(spacerView)
        buttonStackView.addArrangedSubview(button5)
        buttonStackView.addArrangedSubview(button4)
        buttonStackView.addArrangedSubview(firstSeparator)
        buttonStackView.addArrangedSubview(button1)
        buttonStackView.addArrangedSubview(button2)
        buttonStackView.addArrangedSubview(secondSeparator)
        buttonStackView.addArrangedSubview(button3)
        
        // --- Configurar constraints ---
        // Acceder a la lazy var la activa
        NSLayoutConstraint.activate([
            button1WidthConstraint
        ])
    
        
        contentView.addSubview(mainStackView)
        
        NSLayoutConstraint.activate([
            mainStackView.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 20),
            mainStackView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 20),
            mainStackView.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -20),
            mainStackView.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -20)
        ])
    }
    
    // --- FUNCIÓN (Sin cambios, pero ahora accede a lazy vars) ---
    func configure(type: ExperienceType, isElegible: Bool) {
        self.currentType = type // Guardar estado
        
        switch type {
        case .shopping:
            firstSeparator.isHidden = false
            button1.isHidden = false
            button2.isHidden = false
            
            secondSeparator.isHidden = !isElegible
            button3.isHidden = !isElegible
            
            buttonWidthConstraint.isActive = isElegible
            
            spacerView.isHidden = true
            button4.isHidden = true
            button5.isHidden = true
            
        case .default:
            firstSeparator.isHidden = true
            button1.isHidden = true
            button2.isHidden = true
            secondSeparator.isHidden = true
            button3.isHidden = true
            
            spacerView.isHidden = false
            button4.isHidden = false
            button5.isHidden = false
            
            buttonWidthConstraint.isActive = false
        }
        
        updateMainStackLayout()
    }
    
    // MARK: - Helpers
    
    func createDualLabelButton(topText: String, bottomText: String) -> CustomButtonControl {
        let button = CustomButtonControl(type: .dualLabel)
        button.topText = topText
        button.bottomText = bottomText
        button.translatesAutoresizingMaskIntoConstraints = false
        return button
    }
    
    func createImageButton(image: UIImage?, bottomText: String) -> CustomButtonControl {
        let button = CustomButtonControl(type: .imageWithLabel)
        button.image = image
        button.bottomText = bottomText
        button.translatesAutoresizingMaskIntoConstraints = false
        return button
    }
    
    func createSeparator() -> UIView {
        let separator = UIView()
        separator.backgroundColor = .gray
        separator.translatesAutoresizingMaskIntoConstraints = false
        separator.widthAnchor.constraint(equalToConstant: 1).isActive = true
        return separator
    }
    
    @objc func buttonTapped(_ sender: CustomButtonControl) {
        print("Button tapped: \(sender.accessibilityLabel ?? "Unknown")")
    }
    
    // MARK: - Layout Updates
    
    override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
        super.traitCollectionDidChange(previousTraitCollection)

        if traitCollection.preferredContentSizeCategory != previousTraitCollection?.preferredContentSizeCategory ||
           traitCollection.horizontalSizeClass != previousTraitCollection?.horizontalSizeClass ||
           traitCollection.verticalSizeClass != previousTraitCollection?.verticalSizeClass {
            updateMainStackLayout()
        }
    }

    func updateMainStackLayout() {
        let isIPhone = UIDevice.current.userInterfaceIdiom == .phone
        
        let windowScene = self.window?.windowScene
        let isPortrait = windowScene?.interfaceOrientation.isPortrait ?? true
        
        let hasAccessibilityText = traitCollection.preferredContentSizeCategory.isAccessibilityCategory
        
        let isLargeText = isIPhone && isPortrait && hasAccessibilityText && currentType == .shopping
        
        if isLargeText {
            mainStackView.axis = .vertical
            mainStackView.alignment = .fill
            
            // Ocultar separador (solo si es tipo 'shopping')
            if currentType == .shopping {
                firstSeparator.isHidden = true
            }
            
        } else {
            mainStackView.axis = .horizontal
            mainStackView.alignment = .center
            
            // Mostrar separador (solo si es tipo 'shopping')
            if currentType == .shopping {
                firstSeparator.isHidden = false
            }
        }
    }
}








import UIKit

class ViewController: UIViewController {

    private var collectionView: UICollectionView!
    
    // Enum añadido aquí también para que el VC lo conozca
    private enum Section: Int, CaseIterable {
        case contentCell
        case emptyCells
    }
    
    // El enum de tu lógica
    private enum ExperienceType {
        case `default`
        case shopping
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        setupCollectionView()
    }
    
    private func setupCollectionView() {
        let layout = createLayout()
        
        collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: layout)
        collectionView.translatesAutoresizingMaskIntoConstraints = false
        collectionView.dataSource = self
        
        collectionView.register(EmptyCell.self, forCellWithReuseIdentifier: EmptyCell.reuseIdentifier)
        collectionView.register(ContentCell.self, forCellWithReuseIdentifier: ContentCell.reuseIdentifier)
        
        view.addSubview(collectionView)
        
        NSLayoutConstraint.activate([
            collectionView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            collectionView.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor),
            collectionView.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor),
            collectionView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor)
        ])
    }
    
    private func createLayout() -> UICollectionViewLayout {
        let layout = UICollectionViewCompositionalLayout { (sectionIndex, layoutEnvironment) -> NSCollectionLayoutSection? in
            
            guard let sectionKind = Section(rawValue: sectionIndex) else { return nil }
            
            if sectionKind == .emptyCells {
                let itemSize = NSCollectionLayoutSize(
                    widthDimension: .absolute(100),
                    heightDimension: .absolute(100)
                )
                let item = NSCollectionLayoutItem(layoutSize: itemSize)
                
                let groupSize = NSCollectionLayoutSize(
                    widthDimension: .fractionalWidth(1.0),
                    heightDimension: .absolute(100)
                )
                let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])
                group.interItemSpacing = .fixed(10)
                
                let section = NSCollectionLayoutSection(group: group)
                section.interGroupSpacing = 10
                section.contentInsets = NSDirectionalEdgeInsets(top: 10, leading: 20, bottom: 10, trailing: 20)
                return section
                
            } else if sectionKind == .contentCell {
                let itemSize = NSCollectionLayoutSize(
                    widthDimension: .fractionalWidth(1.0),
                    heightDimension: .estimated(100)
                )
                let item = NSCollectionLayoutItem(layoutSize: itemSize)
                
                let groupSize = NSCollectionLayoutSize(
                    widthDimension: .fractionalWidth(1.0),
                    heightDimension: .estimated(100)
                )
                let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])
                
                let section = NSCollectionLayoutSection(group: group)
                section.contentInsets = NSDirectionalEdgeInsets(top: 20, leading: 0, bottom: 0, trailing: 0)
                return section
            }
            
            return nil
        }
        
        return layout
    }
}

// MARK: - UICollectionViewDataSource
extension ViewController: UICollectionViewDataSource {
    
    func numberOfSections(in collectionView: UICollectionView) -> Int {
        return Section.allCases.count
    }
    
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        guard let sectionKind = Section(rawValue: section) else { return 0 }
        
        switch sectionKind {
        case .emptyCells:
            return 30 // Respetando el 30 de tu código
        case .contentCell:
            return 1
        }
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        guard let sectionKind = Section(rawValue: indexPath.section) else {
            fatalError("Unknown section")
        }
        
        switch sectionKind {
        case .emptyCells:
            guard let cell = collectionView.dequeueReusableCell(
                withReuseIdentifier: EmptyCell.reuseIdentifier,
                for: indexPath
            ) as? EmptyCell else {
                fatalError("Could not dequeue EmptyCell")
            }
            return cell
            
        case .contentCell:
            guard let cell = collectionView.dequeueReusableCell(
                withReuseIdentifier: ContentCell.reuseIdentifier,
                for: indexPath
            ) as? ContentCell else {
                fatalError("Could not dequeue ContentCell")
            }
            
            // --- CAMBIO AQUÍ: Llamada a la nueva función ---
            
            // Para probar 'default' (solo muestra button4):
            // cell.configure(type: .default, isElegible: false)
            
            // Para probar 'shopping' sin button3 (elegible: false):
            cell.configure(type: .default, isElegible: true)
            
            // Para probar 'shopping' con button3 (elegible: true):
            // cell.configure(type: .shopping, isElegible: true)
            
            return cell
        }
    }
}




