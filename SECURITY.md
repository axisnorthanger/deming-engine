# Security Policy

## Project status

The Deming Engine is an early-stage design. There is no released code yet, so there are no supported versions to patch. Security reports about the **design** are welcome and taken seriously. Examples: a way around the Airlock, a path from Ring 3 to a privileged action, or a flaw in the handle scheme.

## Reporting a vulnerability

Please **do not open a public issue** for a security problem.

Use GitHub's private vulnerability reporting: go to the **Security** tab of this repository and choose **Report a vulnerability**. Include:

- what you found, and which component it affects (Baffle, Gate 1, Gate 2, Stranger Danger Model)
- a concrete scenario or proof of concept, if you have one
- what an attacker could achieve

You can expect an acknowledgement within 7 days. Reporters are credited unless they ask not to be.

## Scope

In scope: the architecture, diagrams and specifications in this repository, and any code added later.

Out of scope: third-party models, services or frameworks referenced here. Please report those to their maintainers.
