- Feature Name: Not needed
- Start Date: 2018-08-12
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary
[summary]: #summary

Provides a process to be followed when stablising a feature (after implementation is complete), to ensure that features are suitable for inclusion in the stable compiler channel. This process requires positive affirmation of a feature's stability before the feature gate is removed (making the feature usable in the stable compiler channel).

# Motivation
[motivation]: #motivation

Some recent features (notably `match_erganomics`) were introduced to the stable compiler, and shortly after found to have soundness issues. These can be attributed to the small overlap between those who use the nightly toolchain (and experiment with new features regularly) and those who desired this particular feature, leading to a lack of real-world testing of the feature.

# Detailed design
[design]: #detailed-design

- Once implementation of a feature is completed, the tracking issue shall be updated to indicate that the feature is now in the testing phase.
- The relevant team shall decide on the amount of testing that is required for the feature (based on its complexity)
- An announcement shall be made `


This is the bulk of the RFC. Explain the design in enough detail for somebody familiar
with the language to understand, and for somebody familiar with the compiler to implement.
This should get into specifics and corner-cases, and include examples of how the feature is used.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

# Alternatives
[alternatives]: #alternatives

What other designs have been considered? What is the impact of not doing this?

# Unresolved questions
[unresolved]: #unresolved-questions

What parts of the design are still TBD?
