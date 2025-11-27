While both ensure execution on the Main Thread, they behave differently regarding the RunLoop execution order:

Task { @MainActor }: Swift Concurrency is cooperative. If we are already on the Main Actor context, the runtime might execute the code immediately or schedule it as a high-priority job within the current cycle. This creates a race condition where reloadData runs before the TableView has finished its layout calculations.

DispatchQueue.main.async: This explicitly enqueues the block at the end of the Main Thread's current serial queue. It forces a context switch to the next RunLoop iteration.

Conclusion: We are not using async for thread safety (MainActor does that). We are using it for deferral—to guarantee the UI layout pass has completed before we inject the data."
