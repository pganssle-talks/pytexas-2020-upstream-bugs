# Strategy Recap

## Patching upstream 💖

Pros:
- You fix the bug for everyone
- Nothing to maintain afterwards (for you...)
- Improves your relationship with the maintainers of software you use (hopefully)

Cons:
- Delays!
- You have to convince someone to accept your patch (or fix the bug)


## Wrapper functions 🆗

Pros:
- Helps maintain cross-version compatibility
- Easy to remove when the need is done
- Can opportunistically upgrade
- Can roll out immediately

Cons:
- Only works when it's possible to work around the bug.
- Only works for the code you are currently writing.

--

# Strategy Recap

## Monkey patching 🙈

Pros:
- Make targeted global fixes.
- Doesn't complicate packaging or deployment.

Cons:
- Hard to reason about.
- Not likely to be compatible across versions.
- Can cause compatibility problems with other users of the same library.

## Vendoring ☣️

Pros:
- Can unblock dependency resolution issues.
- Isolates any changes from the wider system.

Cons:
- Complicated to implement right.
- Doesn't work well when the vendored package is part of your public interface.
- Tooling support is very weak.

## Maintaining a fork ☢️

Pros:
- Relatively easy to implement in some systems.
- Tools exist for this.

Cons:
- Upstream may not evolve in the same direction as your fork.
- Adds friction with upgrades.
- Compatibility becomes a bigger issue the longer it lasts.
