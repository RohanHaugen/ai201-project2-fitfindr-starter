# FitFindr — Starter Kit

This starter kit contains everything you need to begin Project 2.

## What's Included

```
ai201-project2-fitfindr-starter/
├── data/
│   ├── listings.json          # 40 mock secondhand listings
│   └── wardrobe_schema.json   # Wardrobe format + example wardrobe
├── utils/
│   └── data_loader.py         # Helper functions for loading the data
├── planning.md                # Your planning template — fill this out first
└── requirements.txt           # Python dependencies
```

## Setup

```bash
pip install -r requirements.txt
```

Set your Groq API key in a `.env` file (get a free key at [console.groq.com](https://console.groq.com)):
```
GROQ_API_KEY=your_key_here
```

## The Mock Listings Dataset

`data/listings.json` contains 40 mock secondhand listings across categories (tops, bottoms, outerwear, shoes, accessories) and styles (vintage, y2k, grunge, cottagecore, streetwear, and more).

Each listing has: `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.

Load it with:
```python
from utils.data_loader import load_listings
listings = load_listings()
```

## The Wardrobe Schema

`data/wardrobe_schema.json` defines the format your agent uses to represent a user's existing wardrobe. It includes:

- `schema`: field definitions for a wardrobe item
- `example_wardrobe`: a sample wardrobe with 10 items you can use for testing
- `empty_wardrobe`: a starting template for a new user

Load an example wardrobe with:
```python
from utils.data_loader import get_example_wardrobe
wardrobe = get_example_wardrobe()
```

## Where to Start

1. **Read `planning.md` and fill it out before writing any code.**
2. Verify the data loads correctly by running `python utils/data_loader.py`.
3. Build and test each tool individually before connecting them through your planning loop.

Your implementation files go in this same directory. There's no required file structure for your agent code — organize it however makes sense for your design.


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


## Planning Loop

**How does your agent decide which tool to call next?**
It should first run search_listings, as all interactions require an item. If search_listings returns an empty list, it should return an error message and return early. If an item is found, store the most relevant item and proceed to suggest_outfit. It should work well regardless of whether the user has a wardrobe or not, as both provide guides on what the piece of clothing would work well with. It should store the outfit suggestion, then proceed to create_fit_card. It should utilize the outfit suggestion and stored item in order to create a fit card, storing that to return when completed. After that, it should return the information to the user, as after generating the fit card, it should be finished.

---

## State Management

**How does information from one tool get passed to the next?**
In a session, the agent will store important information in variables. The main ones it will use will be results: storing the results of search_listings, selected_item: storing the most relavent result, outfit_suggestion: storing the recommendation for how the outfit should be worn, and fit_card: storing the caption returned by fit card. These will all be within the loop, and each variable will be called as a part of another tool call.
---

## Error Handling

For each tool, describe the specific failure mode you're handling and what the agent does in response.

| Tool | Failure mode | Agent response |
|------|-------------|----------------|
| search_listings | No results match the query |return empty list |
| suggest_outfit | Wardrobe is empty |Suggest general ideas for an outfit matching the item |
| create_fit_card | Outfit input is missing or incomplete |Output a string with a specific error message. Indicate that the outfit was not present. |

## Spec Reflection

<!-- Reflect on how planning.md shaped your implementation.
     Answer both questions with at least 2–3 sentences each. -->

**One way the spec helped you during implementation:**
I was able to follow the spec by completing each tool in sequence and testing each tool to ensure that it worked properly. I was able to layout what I needed as input, what I needed as output, and what needed to be done to transform the input into the output for each tool. I was also able to rely upon the spec in order to inform me about how the inputs relate to each other, that the output for tools 1 and 2 were the inputs for tool 3.
**One way your implementation diverged from the spec, and why:**
One way my implementation diverged from the spec was due to a misunderstanding of the goals of the tools and loop. I initally thought that each tool should be able to be used individually, and that the user should be able to use each tool only once, and so I put that into the spec. However, upon further reading and understanding of the tools, I came to the realization that the requirement of not having all tools run unconditionally was more so about breaking on errors rather than each tool being independent, as the tools are intended to help each other complete the ultimate goal, that being providing recommendations for thrifted outfits and how to use them.
---

## AI Usage

<!-- Describe at least 2 specific instances where you used an AI tool during this project.
     For each: what did you give the AI as input, what did it produce, and what did you
     change, override, or direct differently?

     "I used Claude to help me code" is not sufficient.
     "I gave Claude my Chunking Strategy section from planning.md and asked it to implement
     chunk_text(). It returned a function using a fixed character split. I overrode the
     chunk size from 500 to 200 because my documents are short reviews, not long guides." -->

**Instance 1**
I gave Claude my spec for tool 1 and asked it to create the search_listings() functioon. It returned a function that was obtaining completely nonsensical listing categories that did not correspond to the dict at all. I realized that this was due to me not providing the data types that would be utilized in searching for proper keywords and updated the names of the categories to the proper ones.


**Instance 2**
I gave Claude my spec for tool 3 and asked it to create the create_fit_card() function. It returned a function that did create a fit card, however I found it unsatisfactory due to hashtags and hyphens/m-dashes interspersed throughout the fit card that made the response seem inauthentic and overly extra. I added additional lines to the prompt in order to correct this behaviour I found harmful to the quality of the responses, which roughly fixed the problem.