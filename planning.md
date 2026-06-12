# FitFindr — planning.md

> Complete this document before writing any implementation code.
> Your spec and agent diagram are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Your planning.md will be reviewed as part of your submission.
> Update it before starting any stretch features.

---

## Tools

List every tool your agent will use. For each tool, fill in all four fields.
You must have at least 3 tools. The three required tools are listed — add any additional tools below them.

### Tool 1: search_listings

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
Searches through the listings in order to return the most relevant to the question.
**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `description` (str): the description of the item, i.e. dated graphic tee
- `size` (str): the size of the item, S, M, L, but other methods are also used.
- `max_price` (float): the highest price the item can be. 

**What it returns:**
<!-- Describe the return value — what fields does a result contain? -->
A list of the most relevant listings. It can also be an empty list if nothing is relevant, which should be handled accordingly.
**What happens if it fails or returns nothing:**
<!-- What should the agent do if no listings match? -->
It should return an empty list.
---

### Tool 2: suggest_outfit

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
This tool take an item and the users wardrobe and outputs a recommendation of what to wear
**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `new_item` (dict): The new item, in the form of a specific item being styled for.
- `wardrobe` (dict): The wardrobe of the user, in the form of what clothes they have, can be empty.

**What it returns:**
<!-- Describe the return value -->
Returns a string describing advice for what outfit to wear and why.
**What happens if it fails or returns nothing:**
<!-- What should the agent do if the wardrobe is empty or no outfit can be suggested? -->
If the wardrobe is empty the tool should suggest general styling tips for the item, if the wardrobe isnt empty advise the user what items would pair with the selected item.
---

### Tool 3: create_fit_card

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
Takes the outfit suggestion and the item listing and outputs a short tag explaining what the outfit is and why the user is wearing it, for use in areas such as Instagram or TikTok.
**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `outfit` (str): The string returned from suggest_outfit, it should describe what the item pairs well with.
- `new_item` (dict): The listing for the item being used. Should most likely be obtained from search_listings.

**What it returns:**
<!-- Describe the return value -->
Returns a string with a short tagline for the chosen item.
**What happens if it fails or returns nothing:**
<!-- What should the agent do if the outfit data is incomplete? -->
If the outfit data is incomplete, it should return a descriptive error message as a string.
---

### Additional Tools (if any)

<!-- Copy the block above for any tools beyond the required three -->

---

## Planning Loop

**How does your agent decide which tool to call next?**
<!-- Describe the logic your planning loop uses. What does it look at? What conditions change its behavior? How does it know when it's done? -->
It should first run search_listings, as all interactions require an item. If search_listings returns an empty list, it should return an error message and return early. If an item is found, store the most relevant item and proceed to suggest_outfit. It should work well regardless of whether the user has a wardrobe or not, as both provide guides on what the piece of clothing would work well with. It should store the outfit suggestion, then proceed to create_fit_card. It should utilize the outfit suggestion and stored item in order to create a fit card, storing that to return when completed. After that, it should return the information to the user, as after generating the fit card, it should be finished.

---

## State Management

**How does information from one tool get passed to the next?**
<!-- Describe how your agent stores and accesses state within a session. What data is tracked? How is it passed between tool calls? -->
In a session, the agent will store important information in variables. The main ones it will use will be results: storing the results of search_listings, selected_item: storing the most relavent result, outfit_suggestion: storing the recommendation for how the outfit should be worn, and fit_card: storing the caption returned by fit card. These will all be within the loop, and each variable will be called as a part of another tool call.
---

## Error Handling

For each tool, describe the specific failure mode you're handling and what the agent does in response.

| Tool | Failure mode | Agent response |
|------|-------------|----------------|
| search_listings | No results match the query |return empty list |
| suggest_outfit | Wardrobe is empty |Suggest general ideas for an outfit matching the item |
| create_fit_card | Outfit input is missing or incomplete |Output a string with a specific error message. Indicate that the outfit was not present. |

---

## Architecture

<!-- Draw a diagram of your agent showing how the components connect:
     User input → Planning Loop → Tools (search_listings, suggest_outfit, create_fit_card)
                                                                          ↕
                                                                   State / Session
     Show what triggers each tool, how state flows between them, and where error paths branch off.
     ASCII art, a Mermaid diagram (https://mermaid.js.org/syntax/flowchart.html), or an embedded
     sketch are all fine. You'll share this diagram with an AI tool when asking it to implement
     the planning loop and each individual tool. -->
User query
    │
    ▼
Planning Loop ───────────────────────────────────────────┐
    │                                                    │
    │                                                    │
Query Parsing (Regex)                                    │
    │                                                    │
    │                                                    │
    ├─► search_listings(description, size, max_price)    │
    │       │ if results=[]                              │
    │       ├──► [ERROR] "No listings found..." → return │
    │       │                                            │
    │       │ if results=[item, ...]                     │
    │       ▼                                            │
    │   Session: selected_item = results[0]              │
    │       │                                            │
    ├─► suggest_outfit(selected_item, wardrobe)          │
    │       │                                            │
    │   Session: outfit_suggestion = "..."               │
    │       │                                            │
    └─► create_fit_card(outfit_suggestion, selected_item)│
            │                                            │
        Session: fit_card = "..."                        │
            │                                            └─ error path returns here
            ▼
        Return session
---

## AI Tool Plan

<!-- For each part of the implementation below, describe:
     - Which AI tool you plan to use (Claude, Copilot, ChatGPT, etc.)
     - What you'll give it as input (which sections of this planning.md, your agent diagram)
     - What you expect it to produce
     - How you'll verify the output matches your spec before moving on

     "I'll use AI to help me code" is not a plan.
     "I'll give Claude my Tool 1 spec (inputs, return value, failure mode) and ask it to implement
     search_listings() using load_listings() from the data loader — then test it against 3 queries
     before trusting it" is a plan. -->

**Milestone 3 — Individual tool implementations:**
I will give Claude each tool spec individually and ask it to implement each tool. I will first utilize hardcoded inputs in order to test each tool, before using data_loader, ensuring that I test positive and negative cases. 
**Milestone 4 — Planning loop and state management:**
I will give Claude my architecture diagram and my planning loop description and ask it to implement the flow from each tool. I will then test it for errors first using hardcoded information, ensuring that errors are properly handled, before testing that the looping functionality works as intended. I will ensure that it doesn't call each tool unconditionally, but instead properly utilizes them for a given situation.
---

## A Complete Interaction (Step by Step)

Write out what a full user interaction looks like from start to finish — tool call by tool call. Use a specific example query.

**Example user query:** "I'm looking for a vintage graphic tee under $30. I mostly wear baggy jeans and chunky sneakers. What's out there and how would I style it?"

**Step 1:**
<!-- What does the agent do first? Which tool is called? With what input? -->
The agent will first utilize it's search tool by calling search_listings("vintage graphic tee", "M", 30). The input is the description of the tee, the max price of the tee, and the general size of the user.  It will output several clothes that are recommended to the user.
**Step 2:**
<!-- What happens next? What was returned from step 1? What tool is called now? -->
Next, the model will use its suggest_outfit tool, using the top result from the previous tool which will probably be lst_006, and the users wardrobe containing the baggy jeans and chunky sneakers. The tool will return a recommendation for an outfit.
**Step 3:**
<!-- Continue until the full interaction is complete -->
After that, the agent will utilize the create_fit_card tool to output a fit card for use in Instagram or other such places.It will take the string from suggest_outfit and the item from search_listings as input, and output the short fit card as a string for output.
**Final output to user:**
<!-- What does the user actually see at the end? -->
The user will see the pieces of clothing found from searching, the recommendation for what clothes to wear, and the fit card that they can use if they wish to.