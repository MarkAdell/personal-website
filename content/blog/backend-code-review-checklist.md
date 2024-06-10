---
title: Backend Code Review Checklist
date: 2024-03-30
tags: ["software engineering"]
ShowToc: false
showTopProgressBar: true
---

I have put together this checklist, which I believe will be applicable to most backend code reviews.

Some of these checks, such as Code Style, should ideally be enforced and detected in the CI pipeline. However, I have included them here for the sake of completeness.

You can use this checklist as a starting point and customize it to suit your specific needs.

## Code Style

- Verify that the code adheres to the agreed-upon coding style guidelines.

## Code Maintainability

- Verify that the code adheres to the clean code principles (or any other agreed-upon principles).

## Requirements

- Verify that the code fulfills the specified requirements.
- Verify that new code doesn't break any existing functionality.

## API Design

- Verify that any new APIs adhere to the agreed-upon API design guidelines.

## Documentation and Comments

- Verify that complex logic or non-obvious decisions are covered by clear comments.
- Verify that any required internal or external code documentation is provided, depending on your agreed-upon documentation processes.

## Error Handling

- Verify that exceptions are handled correctly and that error messages are informative.

## Security

- Verify that inputs are validated properly.
- Verify that sensitive data (passwords, tokens) are securely stored and aren't leaked to logs.
- Examine the code for potential security vulnerabilities, such as SQL injection or authentication issues.

## Dependencies

- Verify that dependencies are up-to-date and don't have known security vulnerabilities.
- Verify that any breaking changes are handled when updating dependencies.

## Logging

- Verify that critical places in the code are covered by logs that are useful for debugging.
- Verify that logging adheres to the agreed-upon logging guidelines.

## Testing

- Verify that the code is covered by the appropriate types of automated tests.

## Performance

- Evaluate the code for performance issues (memory, CPU, network).
- Verify that database queries are optimized.

## Version Control

- Verify that the agreed-upon version control workflow and practices are followed.

## Spelling

- Verify that the spelling is correct, as this makes the code more searchable.

# Conclusion

I hope you found this checklist useful. Please feel free to suggest additional checks that you think are necessary.
