- Feature Name: `move_trait`
- Start Date: 2015-04-30
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary

Add a new trait similar to `Sized` that indicates that a type can be moved using a `memcpy` call.

# Motivation

While the rule that all types are trivially movable is nice for library developers, it means that there is no way of doing complex or efficient memory tricks, such as having a Gc type that knows where its pointers live, or saving a raw pointer to an object.

# Detailed design

* Create a new built-in marker trait `Move` implemented for all types by default, and as a default bound generics.
* A type which has opted out of the `Move` trait (or that contains such a type) cannot be moved out of its first memory location
 * This still allows returning such types, via a hidden output pointer.
* Introduce a new library trait `Relocate` which provides a canonical method for moving a non-`Move` object to another memory location
```rust
trait Relocate
{
	fn relocate(self) -> Self;
}
```

## Example
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
