Hey [Name], I know the Task { @MainActor ... DispatchQueue.main.async } combo looks a bit redundant, but here is why we need both:

Task { @MainActor }: Ensures we switch from the background thread (where the fetch happened) to the Main Thread safely.

DispatchQueue.main.async: This is the trick. It doesn't just run it on the main thread; it queues it at the end of the current RunLoop.

Why it fixes it: By moving the reloadData to the end of the current execution line, we let UIKit finish any pending layout or setup operations first. If we don't do this, reloadData might be called while the view is still initializing or in a 'dirty' state, causing the update to be ignored."
