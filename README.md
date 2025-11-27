Entiendo perfectamente tu tentación. Poner un DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) suele ser el "parche rápido" cuando sentimos que la UI no está lista para recibir datos.

Sin embargo, no te recomiendo usar tiempos fijos (delays) por dos razones:

Es impredecible: En un iPhone 14 Pro, 0.1 segundos es mucho tiempo; en un iPhone XR con batería baja, 0.5 segundos puede no ser suficiente.

Experiencia de usuario: Si el usuario entra rápido, verá el contenido "saltar" (pop-in) justo cuando se cumple el tiempo.

La solución correcta no es esperar tiempo, sino esperar al siguiente ciclo de dibujo (RunLoop).
