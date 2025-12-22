---
title: Domain Modeling Made Functional
publish: false
tags: [ddd]
created: 2025-06-11T00:19:26+09:00
modified: 2025-12-22T20:10:50+09:00
---

# Domain Modeling Made Functional

## Understanding the Domain

### Functional Architecture

- Domain Object
	- an object designed for use only within the boundaries of a context
- Data transfer object(DTO) 
	- an object designed to be serialized and shared between contexts
- Shared Kernel - 
- Anti Corruption layer(ACL) 
	- a component that translates concepts from one domain to another in order to reduce coupling and allow domains to evolve independently
- Persistence Ignorance
	- the domain model should not contain any awareness of databases or other persistence mechanisms

## Modeling the Domain

## Implementing the Model