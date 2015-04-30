- Feature Name: `move_trait`
- Start Date: 2015-04-30
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary

Add a new trait similar to `Sized` that indicates that a type can be moved using a `memcpy` call.

# Motivation

The motivation of this feature is to allow unsafe code to assume that an instance of a type does not change memory addresses without it being informed.
_Personal_ The usecase that caused this RFC was a case where borrow-checker freezing of a type wasn't possible, and using hidden heap allocations would be too expensive. Having a stack-allocated "Sleep object" to control a kernel thread sleeping on a set of events (e.g. interrupt, timer, mutex, etc) which hands out pointers to itself so those event sources can tell it to wake.
_(TODO: Check this with someone who knows about it) Another use could be as part of a GC system, prevening GC objects from being moved without the engine being told_

# Detailed design

* Create a new built-in marker trait `Move` implemented for all types by default.
 * This trait will act similar to `Sized` in that it is an opt-out bound on generics.
 * Like `Send` and `Sync`, it will have a `impl Move for .. {}` implementation applying to all types by default.
* A type which has opted out of the `Move` trait (or that contains such a type) cannot be moved out of its memory location.
 * This requires that the type always (to the user code) reside in memory.
* Introduce a new library trait `Relocate` which provides a canonical method for moving a non-`Move` object to another memory location
```rust
trait Relocate
{
	fn relocate(self) -> Self;
}
```

## Example 1: Storing a pointer to an instance

This trivial example maintains a global pointer (unsynchronised for brevity) to the instance of `NoMove` and updates this pointer when it relocates.

```rust
struct NoMove
{
	inner: u32,
}
impl !::std::marker::Move for NoMove {}

static mut S_GLOBAL_POINTER: Option<*const NoMove> = None;

impl NoMove
{
	fn new() -> NoMove
	{
		let rv = NoMove { inner: 0 };
		unsafe {
			S_GLOBAL_POINTER = Some(&rv as *const _);
		}
		rv
	}
}
impl ::std::marker::Relocate for NoMove
{
	fn relocate(self) -> NoMove {
		// Move the contents of self into a new instance
		let rv = NoMove { inner: self.inner };
		unsafe {
			// Forget the original instance (prevent the destructor from running)
			::std::mem::forget(self);
			// Update the unsafe pointer
			S_GLOBAL_POINTER = Some(&rv as *const _);
		}
		rv
	}
}
impl ::std::ops::Drop for NoMove
{
	fn drop(&mut self) {
		// Clear the saved pointer
		unsafe {
			S_GLOBAL_POINTER = None;
		}
	}
}

fn main()
{
	// New instance
	let nm = NoMove::new();
	
	// Move elsewhere
	let new_nm = nm.relocate();
	
	//{
	//	// Attempt to move trivially
	//	let tmp = new_nm;	// This will not compile
	//}
}
```

# Drawbacks

* This feature does substantially alter one of rust's core features (trivial moves)
* Complex to implement, as it requires a level of optimisation to behave as expected.
 * A local instance that is going to be returned, must be stored within the return pointer from construction
 * (optional) By-value methods should use a hidden pointer, instead of copying to the argument stack

# Alternatives

This feature is mostly a "nice to have", as it allows memory-efficient algorithms by forcing user code to inform the object when it is moved.

# Unresolved questions

What parts of the design are still TBD?
