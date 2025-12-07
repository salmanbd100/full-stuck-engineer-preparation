# Presentation & Public Speaking

Master technical presentations, team demos, and showcase skills for senior engineering roles.

## 📚 Overview

Senior engineers frequently present:
- **Technical demos** to stakeholders
- **Architecture decisions** to teams
- **Project updates** to leadership
- **Knowledge sharing** (lunch & learns)
- **Conference talks** (optional but valuable)

Strong presentation skills demonstrate leadership readiness and communication mastery.

## 🎯 Presentation Structure

### The 3-Act Framework

**Act 1: Hook (10%)**
- Grab attention
- State the problem/topic
- Preview what you'll cover

**Act 2: Content (75%)**
- Main points (3-5 max)
- Examples and demonstrations
- Data and evidence

**Act 3: Conclusion (15%)**
- Summarize key takeaways
- Call to action
- Q&A

---

## 📖 Types of Technical Presentations

### 1. Project Demo (10-15 minutes)

**Structure:**
```
1. Context (2 min)
   "Our dashboard had 4-second load times, causing 30% bounce rate..."

2. Solution (3 min)
   "I implemented three optimizations: code splitting, lazy loading,
    and React.memo..."
   [Show before/after metrics]

3. Demo (5 min)
   [Live demonstration or recorded video]
   "Let me show you the improved user experience..."

4. Impact (2 min)
   "Load time: 4s → 1.6s (60% improvement)
    Bounce rate: 30% → 12%
    User satisfaction: +25%"

5. Q&A (3 min)
```

**Example Opening:**
```
"Hi everyone! Today I'm excited to share the performance improvements
we made to our dashboard. Three months ago, our users were experiencing
4-second load times, which was causing frustration and a 30% bounce
rate. I'll show you how we reduced that to under 2 seconds and the
impact it had on our key metrics. Let's dive in..."
```

---

### 2. Architecture Presentation (20-30 minutes)

**Structure:**
```
1. Problem Statement (3 min)
   "We're designing the architecture for our new microservices platform..."

2. Requirements (5 min)
   "We need to support 10M users, 100k requests/second, <200ms latency..."

3. Proposed Architecture (12 min)
   [Draw diagram or show slides]
   "Here's the high-level architecture. We have three main layers..."
   Walk through components, data flow, key decisions

4. Trade-offs & Alternatives (5 min)
   "We considered monolith vs microservices. Here's why we chose
    microservices for our use case..."

5. Next Steps (2 min)
   "Phase 1: Build core services (Q1)
    Phase 2: Migration (Q2)
    Phase 3: Optimization (Q3)"

6. Q&A (5 min)
```

**Slide Structure:**
- Slide 1: Title + Your name
- Slide 2: Problem/Context
- Slide 3-4: Requirements
- Slide 5-8: Architecture (diagrams, components)
- Slide 9-10: Trade-offs
- Slide 11: Timeline/Next steps
- Slide 12: Thank you + Q&A

---

### 3. Knowledge Sharing / Lunch & Learn (30-45 minutes)

**Example Topics:**
- "Introduction to React Hooks"
- "How We Achieved 99.99% Uptime"
- "Understanding the Event Loop"
- "Best Practices for Code Reviews"

**Structure:**
```
1. Introduction (5 min)
   - Your background with the topic
   - Why it matters
   - What attendees will learn

2. Core Content (25 min)
   - 3-5 main points
   - Live coding demos
   - Real examples from your work

3. Best Practices (5 min)
   - Common pitfalls
   - Recommended approaches
   - Tools and resources

4. Q&A (10 min)
```

**Example: "Introduction to React Hooks"**
```
Slide 1: Title
"Introduction to React Hooks"
By Salman Rahman

Slide 2: What We'll Cover
- What are hooks and why they exist
- useState and useEffect in depth
- Custom hooks
- Best practices and common mistakes

Slide 3: The Problem (Before Hooks)
- Class components were complex
- Logic reuse was hard (HOCs, render props)
- Lifecycle methods were confusing

Slide 4: Enter Hooks (Solution)
- Functional components with state
- Simpler logic reuse
- More intuitive mental model

Slides 5-10: Deep dive into useState, useEffect
[Live coding examples]

Slides 11-12: Custom Hooks
[Real example from our codebase]

Slide 13: Best Practices
- Rules of hooks
- Common mistakes to avoid
- When to use each hook

Slide 14: Resources
- React docs: react.dev
- My blog post: [link]
- Practice repo: [link]

Slide 15: Q&A
```

---

## 🎤 Delivery Techniques

### Voice & Pace

**Speed:**
- **Too slow:** < 100 words/min (boring, loses attention)
- **Optimal:** 120-150 words/min (conversational)
- **Too fast:** > 180 words/min (hard to follow)

**Practice:**
Record yourself and count words. Aim for 120-150 wpm.

**Volume:**
- Loud enough to be heard clearly
- Vary volume for emphasis
- Increase volume for key points

**Pace Variation:**
```
"Our system handles... [pause] ...10 million requests per day.
[pause for impact]

That's 115 requests per second, 24/7, with 99.99% uptime."
```

**Strategic Pauses:**
- After important points (let it sink in)
- Before answering questions (think time)
- During transitions (signal topic change)

---

### Body Language (Video or In-Person)

**DO:**
✅ Maintain eye contact (look at camera for video)
✅ Smile naturally
✅ Use hand gestures to emphasize
✅ Stand/sit up straight (confident posture)
✅ Move occasionally (not frozen, not pacing)
✅ Nod to show engagement during Q&A

**DON'T:**
❌ Cross arms (defensive)
❌ Fidget or play with objects
❌ Turn your back to audience (when writing on board)
❌ Read slides verbatim
❌ Look down constantly
❌ Rock back and forth

---

### Engaging Your Audience

**Interactive Techniques:**
```
1. Ask rhetorical questions:
   "How many of you have experienced slow dashboard load times?"

2. Quick polls:
   "Quick show of hands - who's used React hooks before?"

3. Check understanding:
   "Does this approach make sense so far?"

4. Invite participation:
   "What other approaches could we consider?"

5. Use examples:
   "Let me show you a real-world example from our code..."
```

**Storytelling:**
```
❌ BORING:
"We optimized the database queries and improved performance."

✅ ENGAGING:
"Three months ago, our CTO called me into her office. 'Salman,' she
said, 'customers are complaining about slow load times.' I ran some
tests and discovered our database queries were taking 2 seconds each.
Here's how I solved it..."
```

---

## 📊 Visual Aids

### Slide Design Principles

**1. One Idea Per Slide**
```
❌ BAD SLIDE:
- Code splitting
- Lazy loading
- Tree shaking
- Minification
- Compression
- Caching
[Wall of text, overwhelming]

✅ GOOD APPROACH:
Slide 1: "Code Splitting"
[Clear diagram + 1 code example]

Slide 2: "Lazy Loading"
[Before/after comparison]

Slide 3: "Impact"
[Single chart showing improvement]
```

**2. Limit Text (6x6 Rule)**
- Maximum 6 bullet points per slide
- Maximum 6 words per bullet
- Use visuals instead of text

**3. Use Diagrams**
```
Instead of describing architecture in text:
- Draw boxes and arrows
- Show data flow visually
- Use colors to group related components
- Label clearly
```

**4. Code Snippets**
```
✅ GOOD:
- Syntax highlighted
- 10-15 lines max per slide
- Focus on relevant part
- Increase font size (18pt+)
- Remove unnecessary details

❌ BAD:
- 50 lines of code
- No syntax highlighting
- Small font (12pt)
- Includes imports and boilerplate
```

**5. Data Visualization**
```
For metrics:
- Use bar charts for comparisons
- Line graphs for trends over time
- Highlight the key insight
- Include labels and units

Example:
"Load Time Improvement"
[Bar chart showing: Before: 4.2s, After: 1.6s]
"62% faster!"
```

---

### Live Demos

**Best Practices:**
```
✅ DO:
- Prepare and test beforehand
- Have a backup (screenshot/video)
- Start from a known good state
- Explain what you're doing as you go
- Zoom in (increase font size)
- Have fallback plan if demo breaks

❌ DON'T:
- Live code without preparation
- Debug on stage (unless that's the point)
- Assume Wi-Fi will work
- Use tiny fonts
- Rush through demo
```

**Demo Script Example:**
```
"Let me show you the app before optimization... [loads page]
Notice it takes about 4 seconds to fully load. The spinner is visible
for a long time, and users are waiting.

Now here's the optimized version... [loads improved version]
It loads in under 2 seconds. Much faster. Let me show you what changed
in the code... [switches to code editor]

Here's the key change: I implemented React.lazy for code splitting...
[explains specific code section]"
```

---

## ⚠️ Handling Q&A

### Answering Questions

**1. Listen Fully**
```
❌ BAD:
Interrupting: "Let me stop you there. The answer is..."

✅ GOOD:
Wait for complete question, then:
"That's a great question. Let me make sure I understand..."
```

**2. Paraphrase**
```
"So you're asking about how we handle edge cases in the caching layer.
Let me explain..."
```

**3. Be Honest**
```
If you don't know:
"That's a great question, and I don't have a definitive answer right
now. But here's what I think... [give your best reasoning]. I'll
research this further and follow up with you."

Better than:
- Making up an answer
- Getting defensive
- Saying "I don't know" and stopping there
```

**4. Handle Challenging Questions**
```
Disagreement:
"I appreciate that perspective. Let me explain why I chose this
approach... [reasoning]. But you're right that [acknowledge their
point]. It's definitely a trade-off."

Off-topic:
"That's an interesting question, but it's a bit outside the scope of
today's topic. I'm happy to discuss it after the presentation, or feel
free to ping me on Slack."

Too technical for audience:
"Let me answer that at a high level first, and then we can dig into
the technical details if folks are interested..."
```

---

### Dealing with Nerves

**Before Presentation:**
- Prepare thoroughly (practice 3-5 times)
- Arrive early to set up
- Deep breathing exercises
- Remember: audience wants you to succeed

**During Presentation:**
- First 2 minutes are hardest - push through
- Make eye contact with friendly faces
- If you make a mistake, acknowledge briefly and move on
- Have water nearby
- Slow down if you notice yourself speeding up

**Emergency Recovery:**
```
If you lose your place:
"Let me recap where we are... [reorient yourself]"

If tech fails:
"While we troubleshoot this, let me explain the concept without the
demo..."

If you blank on a question:
"That's a good question. Let me think about that for a moment..."
[pause, gather thoughts]
```

---

## 📚 Practice Routine

### Weekly Practice (for Upcoming Presentation)

**Week 1: Content Preparation**
- Outline your presentation
- Create slides/visuals
- Write speaker notes

**Week 2: First Rehearsal**
- Practice alone, record yourself
- Time yourself
- Review recording, note issues

**Week 3: Refinement**
- Practice 2-3 more times
- Get feedback from colleague
- Refine based on feedback

**Week 4: Final Prep**
- Practice in actual environment (if possible)
- Test all tech (slides, demos, screen sharing)
- Review Q&A preparation
- Practice opening and closing

### General Skill Building

**Monthly:**
- Give 1 lunch & learn or team demo
- Watch 3 tech talks and analyze presentation style
- Present a technical topic to non-technical friend (test clarity)

**Resources:**
- Watch TED talks for storytelling
- Study conference talks in your field
- Join Toastmasters for public speaking practice

---

## ✅ Presentation Checklist

**Content:**
- [ ] Clear objective/goal
- [ ] Logical flow (intro → content → conclusion)
- [ ] 3-5 main points (not too many)
- [ ] Real examples and demos
- [ ] Data/metrics to support points
- [ ] Strong opening and closing

**Slides:**
- [ ] Visually clean (not cluttered)
- [ ] Large fonts (readable from back)
- [ ] Code snippets highlighted and brief
- [ ] Diagrams clear and labeled
- [ ] Consistent theme/colors

**Delivery:**
- [ ] Practiced 3+ times
- [ ] Timed (fits in allotted time + buffer)
- [ ] Voice clear and paced well
- [ ] Body language confident
- [ ] Prepared for Q&A

**Technical:**
- [ ] Slides tested on presentation device
- [ ] Demos tested (have backup)
- [ ] Screen sharing works
- [ ] Fonts/colors visible on screen
- [ ] Links work (if any)

---

**Related:** [Technical Communication](./01-technical-communication.md) | [System Design Communication](./04-system-design-communication.md) | [Cross-Cultural Communication](./06-cross-cultural-communication.md)

---

**Prepare, practice, present with confidence.** 🎤
