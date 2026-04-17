---
layout: default
title: "Project 1: Documentation Website"
---

## Project 1: Documentation Website

**Competency:** Software Design & Engineering

## Overview

This project is an official documentation website for a product from a software development company I was working for, originally created in October 2023. It is a Docusaurus 3-based static site that uses Node.js, hosted on its own domain on a VPS, and uses `.mdx` files for hosting content for documentation and blog pages.

The project was refactored in early 2026, with the main focus of that enhancement being the implementation of lazy loading for images across the site, in an effort to improve performance and the user experience.

## Technologies Used

- **Docusaurus 3.9** (React-based static site generator)
- **Node.js / JavaScript**
- **MDX**

## Enhancement: Lazy Loading for Images (2026)

In early 2026, I refactored this project to implement **lazy loading for images** across the site. This enhancement improves:

- **Performance** — Images are only loaded when they enter the current view of the user's devices or browser. This reduces the initial page load time and the amount of bandwidth and data used for each initial page load.
- **User Experience** — Pages are now able to render faster, which is especially important on slower data connections or devices. This is because the browser doesn't need to download all images of a page when the page is initially loaded.

I felt that this was an important enhancement to include in my portfolio because it was a recent improvement that I made on my own after discovering that the image loading issue existed and was negatively impacting the user experience of some of our users. I believe that this demonstrates my ability to identify issues related to performance and design in an existing codebase and create and apply solutions to those issues to improve the usuability of a production application being used by real people.

### Code Example

This is an example of how I implemented lazy loading for images in this project.

First, I opened a .mdx file that contained an image and imported the `IdealImage` plugin from Docusaurus:

```javascript
import Image from '@theme/IdealImage';
```

Then, I replaced the existing image tag:

```md
![Example Image](/link/to/example_image.png)
```

with the `IdealImage` component and wrapped it in a div to define a max image width to prevent the image from stretching to fill the screen width and bottom padding to create space between back to back images:

```javascript
<div style={{maxWidth: '1000px', marginBottom: '16px'}}>
  <Image img={require('@site/static/img/link/to/example_image.png')} />
</div>
```

## Skills Demonstrated

- Component-based website design
- Web performance improvement
- Working with modern Web tooling (Docusaurus, Node.js)

[Back to Home](./)
