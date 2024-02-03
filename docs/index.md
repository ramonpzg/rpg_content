---
hide:
  - navigation
  - toc
---

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Your Documentation Title</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css">
</head>
<body class="font-sans bg-gray-100">

  <!-- Hero Section -->
  <section class="bg-blue-500 text-white text-center py-16">
    <div class="container mx-auto">
      <h1 class="text-5xl font-extrabold mb-4">Your Documentation Title</h1>
      <img src="path/to/your/hero-image.jpg" alt="Hero Image" class="mx-auto rounded-lg mb-8 shadow-lg">
      <p class="text-lg">A concise and powerful description of your fake product.</p>
    </div>
  </section>

  <!-- Placeholder Images Section -->
  <section class="py-16">
    <div class="container mx-auto text-center">
      <h2 class="text-3xl font-bold mb-8">Placeholder Images</h2>
      <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-8">
        <!-- Add image placeholders here -->
        <img src="path/to/placeholder-1.jpg" alt="Placeholder 1" class="rounded-lg mb-4 shadow">
        <img src="path/to/placeholder-2.jpg" alt="Placeholder 2" class="rounded-lg mb-4 shadow">
        <img src="path/to/placeholder-3.jpg" alt="Placeholder 3" class="rounded-lg mb-4 shadow">
        <img src="path/to/placeholder-4.jpg" alt="Placeholder 4" class="rounded-lg mb-4 shadow">
      </div>
    </div>
  </section>

  <!-- Buttons to Different Sections -->
  <section class="bg-gray-200 py-16">
    <div class="container mx-auto text-center">
      <h2 class="text-3xl font-bold mb-8">Explore Sections</h2>
      <div class="flex justify-center space-x-4">
        <a href="#section1" class="btn btn-primary">Section 1</a>
        <a href="#section2" class="btn btn-primary">Section 2</a>
        <a href="#section3" class="btn btn-primary">Section 3</a>
      </div>
    </div>
  </section>

  <style>
    /* Custom Styles - Feel free to add more */
    .btn {
      display: inline-block;
      padding: 0.75rem 1.5rem;
      font-size: 1rem;
      font-weight: 600;
      text-align: center;
      text-decoration: none;
      cursor: pointer;
      border-radius: 0.25rem;
    }

    .btn-primary {
      background-color: #3490dc;
      /* color: #fff; */
    }
  </style>

</body>




<!-- 
<style>
/*
  over-ride default margins for markdown content in material
*/
.md-main__inner {
  margin-top: 0;
}
.md-main__inner.md-grid {
    margin: 0;
    max-width: 100vw;
}
.md-content__inner {
  margin: 0;
  padding: 0;
}
.md-content__inner::before {
  height: 0;
}
</style> -->

<!-- <div class="primary-swapped-bg-fg m-0">
  <div class="grid-2 items-center pb-2 md-grid">
    <img src="https://staging.svgrepo.com/show/60603/baseball.svg" style="max-height: 12rem; auto;" draggable="false" class="swap-first m-auto">
    <div class="mb-8 mx-2 swap-last">
      <h1> </h1>
      <h2>
        <strong>
            I've been creating educational content for the past 6 years in all sorts of topics and I 
            am currently adding 1 piece of content here per week. So, hopefully every time you come 
            back to this website you'll see more and more cool stuff being added.</br></br>
            Conversely, you can sign up for my newsletter to get notified when I post new content.</strong>
      </h2>
      <p><br></p>
      <a href="https://www.scale.bythebay.io/llm-workshop" class="md-button md-button--primary">
        Get the latest tutorials in your inbox!
      </a>
    </div>
  </div>
</div> -->

<!-- <div class="grid-2 items-center px-2 py-4 md-grid">
  <div class="mb-4 swap-last">
      <h2>Data Content</h2>
      <br>
      <p>The Full Stack brings people together to learn and share best practices across the entire lifecycle of an AI-powered product:
          from defining the problem and picking a GPU or foundation model to production deployment and continual learning
          to user experience design.
      </p>
  </div>
  <img src="images/full_stack_description.png" class="swap-first">
</div> -->

<!-- <div class="primary-swapped-bg-fg">
  <div class="grid-2 items-center py-4 px-2 md-grid">
    <a href="llm-bootcamp"><img src="llm-bootcamp/opengraph.png"></a>
    <div class="mb-4">
        <h2>Engineering Content <a href="llm-bootcamp">Large Language Models Bootcamp</a>.</h2>
        <br>
        <p>
          Learn best practices and tools for building applications powered by LLMs. </p> <p> Cover the full stack from <a href="llm-bootcamp/prompt-engineering">prompt engineering</a> and <a href="llm-bootcamp/llmops">LLMops</a> to <a href="llm-bootcamp/ux-for-luis">user experience design</a>.
        </p>
    </div>
  </div>
</div>

<div class="grid-2 items-center px-2 py-4 md-grid">
  <a href="course"><img src="images/positioning.png" class="swap-first" draggable="false"></a>
  <div class="swap-last">
      <h2>Other Cool Content<a href="course">Deep Learning Course</a>.</h2>
      <p>
        You've trained your first (or 100th) model, and you're ready to take your skills to the next level.
      </p>
      <p>
          Join thousands from <a href="https://bit.ly/berkeleyfsdl">UC Berkeley</a>,
          <a href="https://bit.ly/uwfsdl">University of Washington</a>, and <a
              href="https://youtube.com/c/FullStackDeepLearning">all over the world</a>
          and learn best practices for building AI-powered products from scratch with deep neural networks.
      </p>
  </div>
</div> -->

<!-- 
<div class="grid cards" markdown>

-   :material-clock-fast:{ .lg .middle } [__Introduction__](./01_setup/)

    ---

    Different sections require different set ups. Check it out to see what you'll need.

    [:octicons-arrow-right-24: Getting started](#)

-   :fontawesome-solid-code:{ .lg .middle } [__Analytics__](./02_analytics/)

    ---

    All topics related to Data Analytics that I've created over the years.

    [:octicons-arrow-right-24: Reference](#)

-   :fontawesome-solid-screwdriver-wrench:{ .lg .middle } __Data Engineering__

    ---

    Change the colors, fonts, language, icons, logo and more with a few lines

    [:octicons-arrow-right-24: Customization](#)

-   :fontawesome-solid-chart-line:{ .lg .middle } __Data Visualization__

    ---

    Material for MkDocs is licensed under MIT and available on [GitHub]

    [:octicons-arrow-right-24: License](#)

-   :fontawesome-solid-flask:{ .lg .middle } __Data Science__

    ---

    Material for MkDocs is licensed under MIT and available on [GitHub]

    [:octicons-arrow-right-24: License](#)

-   :fontawesome-solid-terminal:{ .lg .middle } __DevOps__

    ---

    Install [`mkdocs-material`](#) with [`pip`](#) and get up
    and running in minutes

    [:octicons-arrow-right-24: Getting started](#)

-   :fontawesome-brands-markdown:{ .lg .middle } __Machine Learning Engineering__

    ---

    Focus on your content and generate a responsive and searchable static site

    [:octicons-arrow-right-24: Reference](#)

-   :material-format-font:{ .lg .middle } __Generative AI__

    ---

    Change the colors, fonts, language, icons, logo and more with a few lines

    [:octicons-arrow-right-24: Customization](#)

-   :material-scale-balance:{ .lg .middle } __Software Development__

    ---

    Material for MkDocs is licensed under MIT and available on [GitHub]

    [:octicons-arrow-right-24: License](#)

-   :material-clock-fast:{ .lg .middle } __Programming__

    ---

    Install [`mkdocs-material`](#) with [`pip`](#) and get up
    and running in minutes

    [:octicons-arrow-right-24: Getting started](#)

-   :fontawesome-brands-markdown:{ .lg .middle } __Animation and Motion Graphics__

    ---

    Focus on your content and generate a responsive and searchable static site

    [:octicons-arrow-right-24: Reference](#)

-   :material-format-font:{ .lg .middle } __Communication & Presentation__

    ---
    
    <i class="fa-solid fa-code"></i>  
    
    Change the colors, fonts, language, icons, logo and more with a few lines

    [:octicons-arrow-right-24: Customization](#)

</div> -->