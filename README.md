"The reason we need the RunLoop via DispatchQueue.main.async is to handle The Update Cycle.

UIKit optimizes performance by 'coalescing' layout updates. If we call reloadData() synchronously while the view is still initializing or in the middle of a layout pass (which often happens with hidden TableViews), the system might flag the view as 'needs layout' but ignore the immediate request because it thinks the view isn't ready or visible yet.

By using DispatchQueue.main.async, we are effectively scheduling the update for the next iteration of the RunLoop. This guarantees that:

The current layout pass finishes completely.

The view hierarchy is stable.

The reloadData call executes on a 'clean slate,' forcing the engine to acknowledge the change."
