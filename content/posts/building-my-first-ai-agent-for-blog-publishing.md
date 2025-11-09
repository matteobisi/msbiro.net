---
title: "Building My First AI Agent for Blog Publishing"
date: 2025-11-09T16:11:07Z
tags: [
  "ai", "automation", "devops", "open-source", 
  "productivity", "hugo", "cli"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "How I built an AI-powered automation agent to humanize, verify, and publish blog articles in minutes. A practical journey from chatbot to AI CLI tools."
canonicalURL: "https://www.msbiro.net/posts/building-my-first-ai-agent-for-blog-publishing/"
disableShare: true
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://www.msbiro.net/social-image.png"
    alt: "AI Agent for Blog Publishing"
    caption: "From chatbot to AI CLI: building automation that works"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

AI is part of our daily life, and I'm not afraid to say that I'm using it regularly for personal tasks. Naturally, I keep and respect the confidentiality of data, and I use my knowledge to understand what AI is telling me back; AI without being driven the correct way can produce absolute garbage. 

Now I'm transitioning from chatbot to AI CLI usage. I'm a victim of [Network Chuck's](https://www.youtube.com/@NetworkChuck) enthusiasm, so I wanted to build my first AI agent for publishing content on my personal blog. See below how I did it in minutes.

---

## The Problem: Publishing Articles Takes Time

Writing technical blog posts is one thing; publishing them is another. Every time I finish an article, I go through the same routine:

1. Check if the front matter is correct
2. Verify Hugo builds without errors
3. Review the text and validate my English (I'm not a native speaker, so I use AI to help me polish the language)
4. Commit with a proper message
5. Push to GitHub

It's tedious and error-prone. I've pushed broken articles more times than I'd like to admit.

**I needed automation, but not just any automation**. I wanted an AI agent that could:
- Read my articles and make them sound more human
- Verify everything before pushing
- Handle the git workflow automatically
- Actually understand what it's doing (not just blindly executing commands)

---

## From Chatbot to CLI: Why AI Agents?

I've been using ChatGPT and similar tools for a while now, but there's a fundamental limitation: they live in a browser. You copy-paste code, switch contexts, and lose workflow continuity.

Then I discovered AI CLI tools like GitHub Copilot CLI and others. The difference is profound. Instead of asking questions in a browser, you interact with AI directly in your terminal, where your actual work happens.

The real power comes when you move from simple Q&A to **AI agents**: autonomous workflows that can read files, run commands, make decisions, and execute tasks end-to-end.

That's what I wanted to build.

---

## Building the Publishing Agent

Here's how I approached it, step by step.

### Step 1: Define the Workflow

Before writing any code, I mapped out what the agent needed to do:

1. **Read the article** and analyze it for tone
2. **Check for AI patterns** like:
   - Excessive em-dashes (—) that should be semicolons (;) or periods
   - Overly formal transitions ("Moreover", "Furthermore")
   - Repetitive sentence structures
   - Marketing-style language
3. **Verify front matter** matches the Hugo archetype template
4. **Run Hugo syntax check** to prevent broken builds
5. **Propose changes** and wait for my approval
6. **Commit and push** with proper git hygiene

This workflow ensures articles sound human-written and won't break the CI/CD pipeline.

### Step 2: Create the Archetype Check

My blog uses Hugo with a specific archetype template for consistency. Every article needs:

```yaml
title: "Article Title"
date: YYYY-MM-DDTHH:MM:SSZ
tags: ["tag1", "tag2", ...]
author: "Matteo Bisi"
description: "Clear description"
canonicalURL: "https://www.msbiro.net/posts/article-slug/"
draft: false  # Must be false for publishing
# ... other fields
```

The agent validates this structure before doing anything else. If something's missing or incorrect, it stops and proposes fixes.

### Step 3: Hugo Syntax Verification

This was critical. I never wanted to push a broken article again.

The solution is simple but effective:

```bash
hugo --quiet --renderToMemory
```

This builds the entire site in memory without writing files. If it fails, the agent shows an error and proposes solutions. No broken commits reach the repository.

### Step 4: Humanization Logic

This is where it gets interesting. The agent looks for specific patterns that make text sound AI-generated:

**Pattern 1: Em-dash Overload**
```
Before: AI loves em-dashes—they're everywhere—way too much.
After: AI loves em-dashes; they're everywhere, but that's too much.
```

**Pattern 2: Formal Transitions**
```
Before: Moreover, it's important to note that furthermore...
After: It's also important to remember that...
```

**Pattern 3: List-Heavy Writing**
```
Before: The benefits include:
- Benefit 1
- Benefit 2
- Benefit 3

After: This approach brings several benefits. First, it improves X. 
It also enhances Y, and you'll see better results with Z.
```

The agent doesn't just blindly replace text. It analyzes context and proposes specific changes with before/after examples. I review and approve each change.

### Step 5: Visual Feedback with ASCII Art

Because why not make it fun? When everything succeeds, you see:

```
   _____ _    _  _____ _____ ______  _____ _____ 
  / ____| |  | |/ ____/ ____|  ____|/ ____/ ____|
 | (___ | |  | | |   | |    | |__  | (___| (___  
  \___ \| |  | | |   | |    |  __|  \___ \\___ \ 
  ____) | |__| | |___| |____| |____ ____) |___) |
 |_____/ \____/ \_____\_____|______|_____/_____/ 
```

If something fails:

```
 _____  _____   _____   ____   _____  
| ____||  __ \ |  __ \ / __ \ |  __ \ 
| |__  | |__) || |__) | |  | || |__) |
|  __| |  _  / |  _  /| |  | ||  _  / 
| |____| | \ \ | | \ \| |__| || | \ \ 
|______|_|  \_\|_|  \_\\____/ |_|  \_\
```

Small touches like this make automation more enjoyable.

### Step 6: Git Workflow Automation

Once all checks pass, the agent handles git operations:

```bash
git add content/posts/article-name.md
git commit --signoff -m "added new article: [Article Title]"
git push
```

The `--signoff` flag is important for my workflow; it ensures proper attribution. The commit message format is consistent across all articles.

---

## The Agent in Action

Here's what using the agent looks like:

```
Me: "publish article at content/posts/my-article.md"

Agent: 
✓ Reading article...
✓ Found 3 potential humanization improvements:
  1. Line 45: Replace em-dash with semicolon
  2. Line 67: Simplify "Moreover" → "Also"
  3. Line 103: Break list into narrative

Do you want to apply these changes? (yes/no)

Me: "yes"

Agent:
✓ Changes applied
✓ Front matter validated
✓ Hugo syntax check passed

   _____ _    _  _____ _____ ______  _____ _____ 
  / ____| |  | |/ ____/ ____|  ____|/ ____/ ____|
 | (___ | |  | | |   | |    | |__  | (___| (___  
  \___ \| |  | | |   | |    |  __|  \___ \\___ \ 
  ____) | |__| | |___| |____| |____ ____) |___) |
 |_____/ \____/ \_____\_____|______|_____/_____/ 

✓ Committed: "added new article: My Article Title"
✓ Pushed to remote

Done! 🎉
```

The entire process takes seconds instead of minutes.

---

## What I Learned

Building this agent taught me several things:

### AI Agents Are Different from Chatbots

**Chatbots answer questions. Agents execute workflows. The mental model shift is important!** You're not asking for advice; you're delegating tasks to an autonomous system that can read files, run commands, and make decisions within defined boundaries.

### Context Matters More Than Code

The agent instructions (stored in `ai-publish-agent.md`) are more important than the bash script. The AI reads these instructions and follows them intelligently. You're teaching the agent **how to think about the problem**, not just what commands to run.

### Verification Before Action

The Hugo syntax check is non-negotiable. The agent never pushes code without verifying it builds correctly. This single check has saved me from multiple broken deployments.

### Human Approval Is Essential

The agent proposes changes but doesn't apply them automatically. I review every suggestion. This keeps me in control while still benefiting from automation.

### Documentation Is the Agent

By documenting the workflow clearly in `ai-publish-agent.md`, I essentially created the agent. The AI reads those instructions and executes them. If I want to change the workflow, I just update the documentation.

---

## The Tools I Used

Here's what made this possible:

- **Hugo**: Static site generator for my blog
- **Git**: Version control and publishing workflow
- **Bash**: Simple automation script for git operations
- **Markdown**: Agent instructions and documentation
- **AI CLI**: The actual AI interface in my terminal

The beauty is that it's all open-source and transparent. No black boxes, no vendor lock-in.

---

## Repository Structure

I organized everything in a dedicated `ai-automation` folder:

```
ai-automation/
├── README.md              # How to use the agent
├── ai-publish-agent.md    # Agent workflow instructions
└── publish-article.sh     # Bash helper script
```

This lives outside my blog repository so it doesn't get committed with blog content. It's version-controlled separately so I can evolve the automation over time.

---

## Next Steps

This is just the beginning. Here are improvements I'm considering:

1. **Image optimization**: Automatically compress and optimize images before publishing
2. **Link validation**: Check that all external links are valid
3. **SEO checks**: Verify meta descriptions, titles, and keyword usage
4. **Social media automation**: Post to Twitter/LinkedIn when articles go live

But I'm following my own principle: **battle-tested solutions only**. I'll add features as I actually need them, not because they sound cool.

---

## Key Takeaways

If you're thinking about building your own AI agent:

1. **Start simple**: Solve one problem really well before adding complexity
2. **Document the workflow**: Your documentation becomes the agent's instructions
3. **Verify everything**: Never trust automation blindly; always validate
4. **Keep humans in the loop**: AI proposes, humans approve
5. **Use existing tools**: Bash, git, and markdown are enough for powerful automation

AI agents aren't magic. They're well-documented workflows executed by AI that can read, understand, and act on those instructions.

---

## Conclusion

Building this AI agent took less than an hour, but it will save me countless hours going forward. More importantly, it **improves quality**. I'm less likely to publish broken articles or content that sounds like it came from a robot.

This is what AI should be: a tool that amplifies human capability, not replaces it. I still write the articles. I still make the decisions. The AI just handles the tedious, error-prone parts.

If you're still using AI only in browser chatbots, I encourage you to explore CLI tools and agent workflows. The productivity gains are real, and you maintain full control over the process.

Now, if you'll excuse me, I need to ask my agent to publish this article. 😉

---

## Resources

- [Network Chuck's YouTube Channel](https://www.youtube.com/@NetworkChuck) - Amazing content on tech and automation
- [Hugo Documentation](https://gohugo.io/documentation/) - Static site generator
- [GitHub Copilot CLI](https://github.com/github/copilot-cli?locale=en-US) - AI in your terminal
- [Google Gemini CLI](https://github.com/google-gemini/gemini-cli) - Google’s Gemini AI terminal utility
- My ai-automation repository - (Coming soon, deciding where to host it)

Both AI tools listed above have free usage tiers, but come with certain limits.

