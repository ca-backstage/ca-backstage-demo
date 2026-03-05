# Product

## Purpose
A Backstage developer portal instance (ca-backstage-demo) that serves as an internal developer platform. The end goal stated in the brief: create a Java Spring Boot skeleton template that gets instantiated into a GitHub repository, with a CI/CD pipeline for build, test, and deploy -- all orchestrated through Backstage.

## Problems It Solves
- Standardizes how teams create new services (scaffolding via templates)
- Centralizes software catalog, API docs, TechDocs, and Kubernetes visibility
- Provides self-service project creation with GitHub integration
- Gives a single pane of glass for service ownership, dependencies, and documentation

## Current State
The app is a freshly scaffolded Backstage v1.48.0 instance with default example data. The existing template is a simple Node.js service template. The Java Spring Boot template mentioned in the brief has not been created yet.

## How It Should Work
1. A developer opens the Backstage portal, signs in via GitHub OAuth
2. They go to "Create..." and pick a template (the goal is a Java Spring Boot skeleton)
3. The scaffolder creates a new GitHub repo from the template, including CI/CD pipeline config
4. The new component is automatically registered in the Backstage catalog
5. Teams can view their services, APIs, docs, Kubernetes workloads, and notifications from the portal

## User Experience Goals
- GitHub-based SSO sign-in (already configured)
- One-click project creation from templates
- Browsable software catalog with relationship graphs
- Integrated TechDocs, API docs, and Kubernetes dashboards
- Notification system for scaffolder events and other alerts

## Key Users
- francojeferson (registered in org.yaml)
- leoanjossantos (registered in org.yaml)
- guest user (for development)
