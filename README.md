let colorTop = UIColor(hex: "#001D46")
            let colorBottom = UIColor(hex: "#004692")
            let width = view.frame.width
            let gradientImage = UIImage.gradientImage(
                bounds: CGRect(x: 0, y: 0, width: width, height: 300),
                colors: [colorTop, colorBottom],
                startPoint: CGPoint(x: 0, y: 0),
                endPoint: CGPoint(x: 1, y: 1)
            )
            let textAttributes: [NSAttributedString.Key: Any] = [.foregroundColor: UIColor.white]
            
            let scrollEdgeAppearance = UINavigationBarAppearance()
            scrollEdgeAppearance.configureWithTransparentBackground()
            scrollEdgeAppearance.backgroundEffect = UIBlurEffect(style: .dark)
            scrollEdgeAppearance.backgroundImage = gradientImage
            scrollEdgeAppearance.backgroundImageContentMode = .scaleToFill
            scrollEdgeAppearance.shadowColor = .clear
            scrollEdgeAppearance.titleTextAttributes = textAttributes
            scrollEdgeAppearance.largeTitleTextAttributes = textAttributes
            
            let standardAppearance = UINavigationBarAppearance()
            standardAppearance.configureWithDefaultBackground()
            standardAppearance.backgroundImage = gradientImage
            standardAppearance.backgroundImageContentMode = .scaleToFill
            standardAppearance.backgroundColor = .clear
            standardAppearance.shadowColor = .clear
            standardAppearance.titleTextAttributes = textAttributes
            standardAppearance.largeTitleTextAttributes = textAttributes
            
            // Usamos animate para que el cambio de color sea suave si ya estamos en pantalla
            if let transitionCoordinator = transitionCoordinator {
                transitionCoordinator.animate(alongsideTransition: { _ in
                    self.navigationController?.navigationBar.standardAppearance = standardAppearance
                    self.navigationController?.navigationBar.scrollEdgeAppearance = scrollEdgeAppearance
                    self.navigationController?.navigationBar.compactAppearance = standardAppearance
                    self.navigationController?.navigationBar.tintColor = .white
                })
            } else {
                navigationController?.navigationBar.standardAppearance = standardAppearance
                navigationController?.navigationBar.scrollEdgeAppearance = scrollEdgeAppearance
                navigationController?.navigationBar.compactAppearance = standardAppearance
                navigationController?.navigationBar.tintColor = .white
            }
            
            // Translucent false inicial para el estado expandido
            navigationController?.navigationBar.isTranslucent = false
