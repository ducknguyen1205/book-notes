
Every software system provides two distinct values to its stakeholders: **behavior and structure**. Software developers are responsible for ensuring that both values remain high, though they often focus only on the lesser value, which eventually renders the system valueless.

### **Behavior**

The first value of software is its **behavior**. Programmers are hired to make machines implement requirements and fix bugs, which saves or makes money for stakeholders. However, ==software was invented to be "soft," meaning it should be easy to change==. To fulfill this purpose, when stakeholders change requirements, the difficulty of implementing that change should be **proportional only to the scope of the change, and not to the shape of the change**. The mismatch between the scope of requested changes and the system's "shape" (architecture) causes development costs to grow out of proportion.

### **Architecture**

(The text transitions into discussing the importance of architecture and its value.)

### **The Greater Value**

When comparing function (system works perfectly) and architecture (system is easy to change), architecture provides **the greater value**. This is because if you have a program that works perfectly but is **impossible to change**, it will become useless when the requirements inevitably change. Conversely, a program that does not work but **is easy to change** can be made to work. Systems reach a point of uselessness when the cost of change exceeds the benefit of change. Business managers who ask for a change and find the estimated cost prohibitively high are likely to be furious that developers allowed the system to become so impractical to modify.

### **Eisenhower’s Matrix**

President Dwight D. Eisenhower identified two types of problems, the urgent and the important, noting that:

> **I have two kinds of problems, the urgent and the important. The urgent are not important, and the important are never urgent**.

![[Pasted image 20251213100640.png]]
**Figure 2.1 Eisenhower matrix**

Applying this matrix to software:

1. The first value of software, **behavior, is urgent but not always particularly important**.
2. The second value of software, **architecture, is important but never particularly urgent**.

The four priorities for action are ranked as follows:

1. **Important and urgent**.
2. **Important and not urgent**.
3. **Urgent and not important**.
4. **Not urgent and not important**.

The common mistake is elevating **Urgent but not important** features (position 3) to the level of **Important and urgent** features (position 1), leading to the important architecture being ignored in favor of unimportant features. ==Since business managers are not equipped to evaluate the importance of architecture, the software development team bears the **responsibility to assert the importance of architecture over the urgency of features**.==
#### Quote explanation
The statement, attributed to President Dwight D. Eisenhower, is a guiding principle for prioritizing different types of problems, which the source applies directly to software development.

In the context of the two values of software—behavior and architecture—the quote helps explain why architecture is often neglected despite its importance:

- **The Urgent Are Not Important:** In software, the first value, **behavior** (making the machine implement requirements and fix bugs), is considered **urgent**, but not always particularly **important** in the grand scheme of the system's longevity. Developers and business managers often fail to separate features that are urgent but not important from those that are truly urgent and important, leading to the urgent, but less critical, features dominating development time.
- **The Important Are Never Urgent:** The second value of software, **architecture** (the structure that makes the system easy to change), is considered **important**. However, **architecture is never particularly urgent**. Because architecture is not urgent, it is often ignored in favor of the urgent, unimportant features.

The core meaning is that people, including software developers, tend to address immediate demands (urgent behavior) while neglecting the factors that ensure long-term success and viability (important architecture).

The four priorities for action derived from this matrix rank **Important** concerns (architecture) higher than purely Urgent ones (features):

1. Important and urgent.
2. **Important and not urgent** (Architecture is primarily here).
3. Urgent and not important (Behavior/features often fall here).
4. Not urgent and not important.

The sources emphasize that it is the **responsibility of the software development team to assert the importance of architecture over the urgency of features** since business managers are not equipped to evaluate the importance of the system's structure. Allowing architecture to be ignored will eventually make change practically impossible for the system, leading to its uselessness.

### **Fight for the Architecture**

Fulfilling this responsibility entails wading into a **struggle** (or fight) alongside other stakeholders. Developers are stakeholders who must safeguard the software's structure, as this is part of their duty and the reason they were hired. Software architects, specifically, must focus on structure to create an architecture that allows features and functions to be easily developed, easily modified, and easily extended. If architecture is prioritized last, the system will become ever more costly to develop, and eventually, **change will become practically impossible for part or all of the system**. Allowing this to happen means the software development team **did not fight hard enough for what they knew was necessary**.