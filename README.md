import UIKit

class NavigationInterceptor: NSObject, UINavigationControllerDelegate {
    
    // MARK: - Propiedades de Almacenamiento Inteligente
    
    // Usamos dos variables para manejar la memoria de forma segura.
    // 1. Si el delegado original es un objeto "huérfano" (helper), lo retenemos FUERTE para que no crash.
    private var strongOriginalDelegate: UINavigationControllerDelegate?
    
    // 2. Si el delegado original es el propio NavController o un VC, lo guardamos DÉBIL para evitar ciclos.
    private weak var weakOriginalDelegate: UINavigationControllerDelegate?
    
    // Propiedad computada para acceder al delegado sin importar cómo se guardó
    var originalDelegate: UINavigationControllerDelegate? {
        return strongOriginalDelegate ?? weakOriginalDelegate
    }
    
    // Tu closure para inyectar lógica
    var myWillShowCallback: ((UIViewController) -> Void)?
    
    // MARK: - Inicializador
    
    init(originalDelegate: UINavigationControllerDelegate?) {
        super.init()
        
        // Lógica para decidir si guardar Strong o Weak
        if let originalDelegate = originalDelegate {
            // Si el delegado es un UIViewController (o UINavigationController),
            // ya está retenido por la jerarquía de vistas. Usamos WEAK para evitar ciclos.
            if originalDelegate is UIViewController {
                self.weakOriginalDelegate = originalDelegate
            } else {
                // Si es una clase helper pura (NSObject), probablemente nadie más la retenga.
                // Usamos STRONG para evitar el crash "unrecognized selector sent to instance".
                self.strongOriginalDelegate = originalDelegate
            }
        }
    }
    
    // MARK: - Intercepción (El único método que "robamos")
    
    func navigationController(_ navigationController: UINavigationController, willShow viewController: UIViewController, animated: Bool) {
        
        // 1. Ejecutar la lógica del delegado ORIGINAL primero (simulando super.willShow)
        originalDelegate?.navigationController?(navigationController, willShow: viewController, animated: animated)
        
        // 2. Ejecutar TU lógica personalizada después
        myWillShowCallback?(viewController)
    }
    
    // MARK: - Message Forwarding (La Magia)
    
    // Estas funciones deben ser 'nonisolated' para cumplir con NSObject en Swift 6+.
    // Usamos 'MainActor.assumeIsolated' porque sabemos que UIKit corre en el hilo principal.
    
    override nonisolated func forwardingTarget(for aSelector: Selector!) -> Any? {
        return MainActor.assumeIsolated {
            // Si nosotros no implementamos el método, pásaselo al delegado original
            return originalDelegate
        }
    }
    
    override nonisolated func responds(to aSelector: Selector!) -> Bool {
        // 1. ¿Lo implementa el Interceptor (ej. willShow)?
        if super.responds(to: aSelector) { return true }
        
        // 2. Si no, ¿lo implementa el delegado original?
        return MainActor.assumeIsolated {
            return originalDelegate?.responds(to: aSelector) ?? false
        }
    }
}
