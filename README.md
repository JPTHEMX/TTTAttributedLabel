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
        
        let attributes: [NSAttributedString.Key: Any] = [
            .font: UIFont.preferredFont(forTextStyle: .caption2),
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

class ViewController: UIViewController {

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
    
    var firstSeparator: UIView?

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        setupLayout()
    }

    func setupLayout() {
        
        let button1 = createDualLabelButton(topText: "100", bottomText: "Title Label One")
        let button2 = createDualLabelButton(topText: "200", bottomText: "Title Label Two")
        let button3 = createImageButton(image: UIImage(systemName: "photo"), bottomText: "Title Label Three")
        
        button1.addTarget(self, action: #selector(buttonTapped(_:)), for: .touchUpInside)
        button2.addTarget(self, action: #selector(buttonTapped(_:)), for: .touchUpInside)
        button3.addTarget(self, action: #selector(buttonTapped(_:)), for: .touchUpInside)
        
        firstSeparator = createSeparator()
        buttonStackView.addArrangedSubview(firstSeparator!)
        buttonStackView.addArrangedSubview(button1)
        buttonStackView.addArrangedSubview(button2)
        buttonStackView.addArrangedSubview(createSeparator())
        buttonStackView.addArrangedSubview(button3)
        
        NSLayoutConstraint.activate([
            button1.widthAnchor.constraint(equalTo: button2.widthAnchor),
            button2.widthAnchor.constraint(equalTo: button3.widthAnchor)
        ])
    
        
        view.addSubview(mainStackView)
        
        NSLayoutConstraint.activate([
            mainStackView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            mainStackView.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor, constant: 20),
            mainStackView.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor, constant: -20)
        ])
        
        updateMainStackLayout()
    }
    
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
        
        let windowScene = view.window?.windowScene
        let isPortrait = windowScene?.interfaceOrientation.isPortrait ?? true
        
        let hasAccessibilityText = traitCollection.preferredContentSizeCategory.isAccessibilityCategory
        
        let isLargeText = isIPhone && isPortrait && hasAccessibilityText
        
        if isLargeText {
            mainStackView.axis = .vertical
            mainStackView.alignment = .fill
            firstSeparator?.isHidden = true
        } else {
            mainStackView.axis = .horizontal
            mainStackView.alignment = .center
            firstSeparator?.isHidden = false
        }
    }
    
    override func viewWillTransition(to size: CGSize, with coordinator: UIViewControllerTransitionCoordinator) {
        super.viewWillTransition(to: size, with: coordinator)
        
        coordinator.animate(alongsideTransition: { _ in
            self.updateMainStackLayout()
        })
    }
}
