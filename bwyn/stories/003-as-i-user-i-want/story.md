# Story: Create a new Jira project

- **Story**: 003-as-i-user-i-want
- **Created**: 2026-10-27
- **Status**: Draft

## Story
As a user, I want to create a new project, so that I can organise my work within the system.

## Acceptance tests
1. **Given** a free user who has 0 existing projects, **When** they create a new project, **Then** the project is saved and appears in their project list.
2. **Given** a free user who already has 30 projects, **When** they attempt to create a 31st project, **Then** the system rejects the request and displays a message indicating the 30-project limit has been reached.
3. **Given** a premium user who already has 70 projects, **When** they attempt to create a 71st project, **Then** the system rejects the request and displays a message indicating the 70-project limit has been reached.
4. **Given** a premium user who has 69 existing projects, **When** they create a new project, **Then** the project is saved and appears in their project list.

## Business rules and constraints
- Free users may hold a maximum of 30 projects. (Driver: Core Functionality — assumed from request description.)
- Premium users may hold a maximum of 70 projects. (Driver: Core Functionality — assumed from request description.)
- Projects are stored in PostgreSQL. (ADR 0001)
- This story depends on AE-0004 being complete before it can be built and shipped.
- The story must be small enough for one developer to build, test and ship without handoff. (Driver: Constraints)

## Out of scope
- Editing or deleting an existing project.
- Defining what fields or configuration a project contains beyond creation.
- Upgrading a user from free to premium tier.
- Any UI beyond the minimum needed to trigger and confirm creation.
