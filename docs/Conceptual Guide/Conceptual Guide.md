# Conceptual Guide

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

Components