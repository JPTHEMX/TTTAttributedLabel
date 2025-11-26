I’ve confirmed that the swizzling implementation is unrelated to this bug.

I paired with Paul to test it, and the behavior seems random/intermittent. It works fine on the first login, but fails on subsequent logins.

The root cause: We are calling reloadData, but it happens while the carousel is not currently visible on screen. That's why the update isn't reflected until the user scrolls and forces a redraw. I'm working on a fix for this timing issue."
