---
date: "2024-06-23"
tags: ["Go", "Conferences"]
title: "What Happened at GopherCon Europe 2024"
---

{{< figureCupper
img="gophercon_eu_wall.jpg" 
caption="GopherCon EU wall" 
command="Resize" 
options="750x" >}}

GopherCon Europe 2024 took place from June 17th to 20th in Berlin, at the Alte Münze Berlin. The venue featured two spacious rooms for Track 1 and Track 2, an outside area, and the Lichthof, which housed breakfast and sponsor booths. This area became the hub for networking and social interactions throughout the conference. I attended on June 19th and 20th and watched about 13 talks and discussions. While all were interesting and relevant, the following are the ones that stood out the most.

## Day 1 - Track 1

### The Business of Go

The first day introduced us to Cameron Balahan, Go's Product Manager. PMs are responsible for delivering the right experience to the right market at the right time. Cameron shared insights into the business of Go and strategies to identify the right timing and experience to deliver, focusing on providing value for both Google and Go users. He concluded by discussing Go's role in Generative AI (GenAI), stating that Go is ideal for the scalable production components of AI-powered products due to its:

- Reliability and security
- Performance and scalability
- Low maintenance and operational cost

### LangChainGo

Travis Cline (tmc), the creator of LangChainGo, provided a look into the major suppliers of Generative AI and the LangChain ecosystem. LangChain is a framework that simplifies the creation of applications powered by large language models (LLMs). Originally written in Python, LangChain also offers a JavaScript/TypeScript version. Recognizing Go's potential in GenAI, tmc developed LangChainGo to extend the ecosystem to Go applications. He demonstrated the framework's capabilities through real-time demos and encouraged those interested in Go and GenAI to contribute to the project. For more details, check out [LangChainGo's documentation](https://tmc.github.io/langchaingo/docs/).

### Go Time Podcast

Later in the day, we enjoyed a live recording of the Go Time podcast. Mat Ryer hosted a game called "Gopher's Say," involving speakers and guests from the Go community. Attendees had previously answered a questionnaire about various topics related to Golang and software engineering, and the game players had to guess the most popular responses. Some of the questions included:

- What is the first Go function that Gophers think of?
- What do Gophers like to drink while working?
- What websites do coding Gophers use the most?

The questions and answers will be available in the Go Time podcast episode. If you're curious, you can listen to it on their [Spotify](https://open.spotify.com/show/2cKdcxETn7jDp7uJCwqmSE?si=a17f7512dab544ed) or find them in my [Playlist](/playlist).

## Day 2

### Technical Documentation

On the second day, Hila Fish, a DevOps Tech Lead, emphasized the importance of good technical documentation in increasing developer velocity. Effective technical docs, which are simple, intuitive, and easy to read, help with:

- Developer self-service enablement
- Avoiding bottlenecks
- Providing visibility into completed work
- Understanding the reasoning behind processes

Hila shared valuable tips for crafting high-quality technical documentation. Key takeaways include:

- **Organize with a Table of Contents**: Ensure important highlights are easily accessible.
- **Use Concise Language**: Opt for short words and clear sentences.
- **Understand Your Audience**: Determine if the documentation is for internal maintainers or external users.
- **Choose the Right Documentation Type**: Decide whether it will be documentation as code or a dedicated knowledge base.
- **Address Audience Needs**: Provide relevant concepts such as information, background, explanations, and reasoning, as well as practical "how-to" tasks.
- **Seek Feedback**: Share your documentation to gather constructive input.

The golden rule for technical documentation is to provide readers with the information they need and send them back to their task as soon as possible. For more information on this topic, Hila wrote and shared [Technical Writing Tips](https://docs.google.com/document/d/1naq4pq0otqb78hkQ8enJBLd_yHqLeMC93fFJAnaK0Rc/edit) to helps us elevate our technical writing skills.

### Securing Containers Against Known Go Vulnerabilities

Zvonimir Pavlinovic, a Software Engineer at the Go team, presented on securing containers against known Go vulnerabilities. He revealed that approximately 80% of Go binaries are built with unsupported Go versions, which often have vulnerabilities in the Go standard library. Around 55% of containers are vulnerable because of this. Zvonimir demonstrated how to use the scalibr tool to update outdated Go binaries. The tool can be customized to include a detector and extractor that reads a binary's Go version and reports when it is outdated. Check out the [scalibr](https://github.com/google/osv-scalibr) documentation to learn about all its capabilities.

## Conclusion

GopherCon Europe 2024 was a platform for learning, networking, and community building. The event covered a wide range of topics and was not only a great opportunity to learn about the latest developments in Go but also a chance to know and contribute to the community and help shape the future of the language.