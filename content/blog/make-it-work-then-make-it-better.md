---
title: Make It Work, Then Make It Better
date: 2024-11-18
tags: ["software engineering"]
ShowToc: false
showTopProgressBar: true
---

In software engineering, the principle "Make it work, then make it better" highlights a practical yet disciplined approach to building high quality software. It consists of two phases: 

1. The first phase focuses on ensuring the software works as intended, delivering the expected results and meeting the requirements. At this stage, we make the "computer" understand the code.
2. The second phase shifts the focus to humans, refactoring the code for your future self and fellow developers who will read, use, and maintain it.

This doesn't mean ignoring code quality in the first phase; it might take around 30% of the focus, depending on context.

Needless to say, phase two should always be completed **before submitting the code for review** or delivering it to customers.

## Make It Work

**Prioritizing Functionality**: Ensuring the program works as intended. This might involve hardcoded values, inefficiencies, unhandled exceptions, etc., but that's okay for now.

**Speed Over Perfection**: Delivering a working solution quickly allows us to validate our approach before investing time in enhancing it.

**Understanding the Problem**: Writing quick, functional code helps developers gain a better understanding of the problem's details.

## Make It Better

Here are some key areas to focus on:

**Clear Naming**: Assigning clear and descriptive names to variables, functions, and classes that reflect their intent.

**Code Style**: Adhering to coding standards and guidelines to ensure consistency and readability.

**Code Design**: Following established design principles like SOLID to enhance code maintainability.

**Architecture**: Revisiting the architecture decisions to ensure they suit the specific feature requirements.

**Testing**: Writing automated tests of the appropriate type (unit, integration, etc.) to ensure the code behaves as expected and prevents new updates from breaking existing functionality.

**Logging**: Adding meaningful logs to improve visibility and aid debugging.

**Error Handling**: Ensuring exceptions are handled correctly and gracefully to maintain stability and reliability.  

**Comments**: Adding comments where necessary to clarify non-obvious logic.

**Performance**: Ensuring that the code is efficient and scalable as needed.

# Conclusion

"Make it work, then make it better" encourages developers to focus on solving problems pragmatically before striving for perfection. Write the messy first draft, but don't forget to come back and polish it into a masterpiece!

It's very common for the second phase to be ignored altogether, and this is exactly how technical debt accumulates. Always make sure to apply the second phase when building software, unless it's a throwaway prototype. This **at least** ensures that the software quality will not degrade over time, which in my experience is an achievement in itself.

It's worth noting that **this principle isn't limited to programming;** you can apply it to many areas of life. In fact, I used this approach while writing this blog! Here's how it went:
- I wrote a quick draft. (phase 1)
- I used chatGPT to do some enhancements and rephrasings. (phase 2)
- I made several iterations on chatGPT's output to make it simpler and more relevant. (phase 2)
