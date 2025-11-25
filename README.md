
import UIKit

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
    
    // Si el método NO es willShow, le preguntamos a Swift: "¿A quién debería enviarle este mensaje?"
    // Respondemos: "Envíaselo al delegado original".
    override func forwardingTarget(for aSelector: Selector!) -> Any? {
        return originalDelegate
    }
    
    // Para que el sistema sepa que respondemos a lo mismo que el delegado original
    override func responds(to aSelector: Selector!) -> Bool {
        if super.responds(to: aSelector) { return true }
        return originalDelegate?.responds(to: aSelector) ?? false
    }
}



class MiComponente {
    
    var navigationController: UINavigationController?
    
    // Guardamos el interceptor en memoria
    private var interceptor: NavigationInterceptor?
    
    func injectLogic() {
        guard let nav = navigationController else { return }
        
        // 1. Capturamos quien sea que esté manejando la lógica actualmente.
        // Si el CustomNav es su propio delegado, aquí lo guardamos.
        let oldDelegate = nav.delegate
        
        // 2. Inicializamos el interceptor pasándole ese delegado antiguo
        let newInterceptor = NavigationInterceptor(originalDelegate: oldDelegate)
        
        // 3. Definimos TU lógica que ocurrirá DESPUÉS de la original
        newInterceptor.myWillShowCallback = { [weak self] viewController in
            print("✅ 1. Lógica original ejecutada.")
            print("👉 2. Ahora ejecutando mi lógica personalizada.")
            self?.configurarMiComponente(vc: viewController)
        }
        
        // 4. Retenemos y asignamos
        self.interceptor = newInterceptor
        nav.delegate = newInterceptor
    }
    
    func configurarMiComponente(vc: UIViewController) {
        // Tu código aquí
    }
}



