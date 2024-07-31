# Conceptual Guide

---

<aside>
📔  **TABLE OF CONTENTS**

- [Architecture](https://www.notion.so/Conceptual-Guide-86fe1a534e114c928acaec82c2a2e5c1?pvs=21)
    - [High-Level Overview](https://www.notion.so/Architecture-292076c23e564f20afa16ee777cfbe1d?pvs=21)
    - [Components and Modules](https://www.notion.so/Architecture-292076c23e564f20afa16ee777cfbe1d?pvs=21)
        - [User Management](https://www.notion.so/Architecture-292076c23e564f20afa16ee777cfbe1d?pvs=21)
        - [Content Management](https://www.notion.so/Architecture-292076c23e564f20afa16ee777cfbe1d?pvs=21)
        - [Collaboration](https://www.notion.so/Architecture-292076c23e564f20afa16ee777cfbe1d?pvs=21)
        - [Notifications](https://www.notion.so/Architecture-292076c23e564f20afa16ee777cfbe1d?pvs=21)
        - [Integration](https://www.notion.so/Architecture-292076c23e564f20afa16ee777cfbe1d?pvs=21)
    - [Data Flow Diagram](https://www.notion.so/Architecture-292076c23e564f20afa16ee777cfbe1d?pvs=21)
- [Components](https://www.notion.so/Components-1bc2f531b75148a18920c5d26f60d855?pvs=21)
    - [Configuration Options](https://www.notion.so/Components-1bc2f531b75148a18920c5d26f60d855?pvs=21)
        - [Environment Variables](https://www.notion.so/Components-1bc2f531b75148a18920c5d26f60d855?pvs=21)
        - [Application Settings](https://www.notion.so/Components-1bc2f531b75148a18920c5d26f60d855?pvs=21)
        - [Integration Settings](https://www.notion.so/Components-1bc2f531b75148a18920c5d26f60d855?pvs=21)
    - [Database Configuration](https://www.notion.so/Components-1bc2f531b75148a18920c5d26f60d855?pvs=21)
    - [API Keys](https://www.notion.so/Components-1bc2f531b75148a18920c5d26f60d855?pvs=21)
    - [Configuration Best Practices](https://www.notion.so/Components-1bc2f531b75148a18920c5d26f60d855?pvs=21)
</aside>

> Explain the underlying concepts and design principles behind OmAgent, and using OmAgent's features for tasks like video understanding and high-accuracy question answering.
> 

# Architecture

OmAgent as a framework consists of a number of packages

## Omagent-core

This package contains base abstractions of different components and ways to compose them together. The interfaces for core components like encoder, LLMs, node, prompt, tool system and more are defined here. The dependencies are intentionally kept minimal.

## Engine

This package contains customized definition for different personal tasks, including loop, node, tools and more specific tasks.

## Workflows

Here we define the base information the task needs, for your custom task, you may need to provide your specific information about llms, nodes and tools, you will define them in Engine part, but here json file should be write to make config for them.

### Explore More Sections

[Components](https://www.notion.so/Components-1bc2f531b75148a18920c5d26f60d855?pvs=21)