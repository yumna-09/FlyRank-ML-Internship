# Retrospective — FlyRank ML Internship

When I started the FlyRank ML Internship, my main goal was to understand how a machine learning project moves from a problem to something that can support a real decision. I was interested in machine learning, but I was still learning how to connect individual technical steps—data, features, models, evaluation, and recommendations—into one complete workflow.

Throughout the internship, my understanding changed from thinking about machine learning mainly as “training a model” to seeing it as a complete process of problem framing, data understanding, validation, interpretation, and communication.

For my capstone, I focused on the question: **which content pages should a strategist review first when a large content backlog makes manual prioritization difficult?** I worked with 30,000 anonymized content pages and explored search-performance and engagement signals to identify behavioral patterns. I eventually used KMeans clustering to group the pages into seven behavioral archetypes and translated those archetypes into a human-review action queue: Refresh, Boost, Prune, and Monitor.

One of the biggest changes in the project came during validation. My initial KMeans evaluation produced a silhouette score of 0.3763. However, while reviewing the evaluation methodology, I identified an evaluation-leakage issue: the clustering centroids had been fitted and evaluated on the same data. Instead of keeping the higher score because it looked better, I changed the validation approach to hold out entire clients during model fitting. The resulting V2 silhouette score was 0.3046.

This was an important lesson for me because it changed how I think about model evaluation. A higher metric is not automatically a better result if the evaluation design is not trustworthy. In this case, the lower V2 score was more useful because it gave a more credible view of how the clustering approach behaved on unseen clients. This also made me more comfortable with the idea that good ML work sometimes means reporting a weaker-looking result when it is the more honest one.

Another important change was learning to think beyond the model itself. A clustering model produces groups, but a stakeholder needs to know what those groups mean and what they can do with them. Turning the behavioral archetypes into a ranked human-review action queue helped me understand the difference between a technical output and a useful decision-support tool. The final recommendations are deliberately not automatic decisions; they are intended to help a human reviewer decide where to start.

I also learned how to use AI tools more responsibly as part of my workflow. AI helped me with brainstorming, coding support, debugging, documentation, and iteration, but I learned that using AI does not remove the responsibility to understand and verify the work. I had to check the analysis, question unexpected results, identify the evaluation issue, and make decisions about the final methodology and presentation myself.

If I were starting the internship again, I would spend more time early on understanding the evaluation design and defining what would make the result trustworthy before focusing on improving model metrics. I would also think about the final user of the analysis earlier, so that the path from model output to practical action is considered from the beginning.

The three most transferable things I learned are:

1. **Evaluation quality matters more than a good-looking metric.** A trustworthy evaluation is more valuable than an inflated result.
2. **ML is a workflow, not just a model.** Problem framing, data quality, leakage checks, validation, interpretation, and communication are all part of building a useful ML solution.
3. **AI is most useful when I remain in the loop.** AI can accelerate coding and documentation, but I still need to understand, verify, question, and take ownership of the final work.

The next step I would take is to build on this foundation with a project that moves further from analysis toward a usable ML or AI product, while keeping the same focus on validation, transparency, and human decision-making.
