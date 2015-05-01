- Feature Name: `move_trait`
- Start Date: 2015-04-30
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary

Add a new trait similar to `Sized` that indicates that a type can be moved using a `memcpy` call. A type that opts out of this trait cannot be moved trivially, and requires an explict move call for it to change memory location.

# Motivation

The motivation of this feature is to allow unsafe code to assume that an instance of a type does not change memory addresses without it being informed.

## Usecases
* https://github.com/servo/servo/pull/5855 - Currently servo uses a lint to avoid accidentally moving a particular type.
* Allowing unsafe code to assume that an object never changes address (for avoiding heap allocations)
 * The usecase that caused this RFC was a case where borrow-checker freezing of a type wasn't possible, and using hidden heap allocations would be too expensive. Having a stack-allocated "Sleep object" to control a kernel thread sleeping on a set of events (e.g. interrupt, timer, mutex, etc) which hands out pointers to itself so those event sources can tell it to wake.
* _TODO: Possible use as part of a Gc system_


# Detailed design

* Create a new built-in marker trait `Move` implemented for all types by default.
 * This trait will act similar to `Sized` in that it is an opt-out bound on generics, requiring `impl<T: ?Move>` to accept non-`Move` types.
 * Like `Send` and `Sync`, it will have a `impl Move for .. {}` implementation applying to all types by default.
 * To opt-out of the `Move` trait, use a negative impl `impl !Move for .. {}`
* A type which has opted out of the `Move` trait (or that contains such a type) cannot be moved out of its memory location.
 * This requires that the type always (to the user code) reside in memory.
* Introduce a new library trait `Relocate` which provides a canonical method for moving a non-`Move` object to another memory location
```rust
trait Relocate
{
	fn relocate(self) -> Self;
}
```
 * _TODO: Where should this type live? marker half fits, but it's closer in use to Clone_

## Example 1: Storing a pointer to an instance

This trivial example maintains a global pointer (unsynchronised for brevity) to the instance of `NoMove` and updates this pointer when it relocates.

```rust
use core::marker::{Move,Relocate};
use core::ops::Drop;

struct NoMove
{
	inner: u32,
}
impl !Move for NoMove {}

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
impl Relocate for NoMove
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
impl Drop for NoMove
{
	fn drop(&mut self) {
		// Clear the saved pointer
		unsafe {
			S_GLOBAL_POINTER = None;
		}
	}
}

// A generic function that can take an immovable value
fn generic_fcn<T: ?Move+Clone>(val: &T) -> T {
	// Just clone it, for an example
	val.clone()
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
* It introduces fracturing of the type space in the same way as Sized did.
* Complex to implement, as it requires a level of optimisation to behave as expected.
 * A local instance that is going to be returned, must be stored within the return pointer from construction
 * (optional) By-value methods should use a hidden pointer, instead of copying to the argument stack.

# Alternatives

This feature is mostly a "nice to have", as it allows memory-efficient algorithms by forcing user code to inform the object when it is moved.

# Unresolved questions

* Location and naming of the `Relocate` trait
* Specifics of by-value behavior
 * Return values have to not move memory location
 * By-value calls would presumably need to use a hidden pointer to avoid relocating the value.
