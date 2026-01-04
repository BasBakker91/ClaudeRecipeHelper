# Albert Heijn Link Generator

## Project Overview

This project generates Albert Heijn shopping list links from a list of groceries. It takes grocery items as input, matches them against the Albert Heijn product database, and creates shareable shopping list URLs.

## Project Structure

```
AlbertHeijnLinkGenerator/
├── .claude/                 # Claude Code configuration
│   └── settings.local.json  # Local settings for Claude Code
└── CLAUDE.MD               # This file - project documentation for Claude
```

## Development Guidelines

### Code Style
- Follow consistent naming conventions
- Keep functions focused and single-purpose
- Add comments for complex logic

### Security Considerations
- Validate all user inputs
- Sanitize URLs before generation
- Handle errors gracefully

## How It Works

### Data Source
- Product data is loaded from: `supermarkets.json` (local file)
- Filter for Albert Heijn products only

### Input
- User provides a list of grocery items with quantities

### Output
The tool provides two outputs:

1. **Generated Link** - An Albert Heijn shopping list URL provided in two formats:
   - Clickable markdown link: [Click here to add items to your AH shopping list](url)
   - Copyable code block:
   ```
   https://www.ah.nl/mijnlijst/add-multiple?p=417523:1&p=572443:2&p=572083:1&p=490280:1&p=382984:1
   ```

2. **Products Not Found** - A list of items that couldn't be matched to the database, each in its own copyable code block for easy reference

### URL Format Specification
- Base URL: `https://www.ah.nl/mijnlijst/add-multiple`
- Query parameters: `?p={product_code}:{quantity}&p={product_code}:{quantity}...`
- Each product is specified as: `p={product_code}:{quantity}`
- Multiple products are joined with `&`

### Example Output Format

**Shopping List Link:**

[Click here to add items to your Albert Heijn shopping list](https://www.ah.nl/mijnlijst/add-multiple?p=417523:1&p=572443:2&p=572083:1&p=490280:1&p=382984:1)

```
https://www.ah.nl/mijnlijst/add-multiple?p=417523:1&p=572443:2&p=572083:1&p=490280:1&p=382984:1
```

**Products Not Found:**
```
Organic quinoa
```
```
Exotic fruit mix
```
```
Specialty cheese
```

Each product that couldn't be matched gets its own code block for easy copying.

## Features

- Match grocery items to Albert Heijn product codes
- Generate shareable shopping list URLs
- Support quantity specification for each item
- Provide complete recipes with ingredients, instructions, and shopping links

## Getting Started

(To be documented once setup instructions are available)

## Technology Stack

(To be documented once technologies are chosen)

## Notes for Claude

### Workflow
1. User provides a list of grocery items (with quantities)
2. Run parallel Grep searches for all products at once using case-insensitive regex patterns
3. From Grep results, identify the best match for each item (prefer AH brand, match size if specified)
4. Extract product codes from the "l" field using Bash grep (e.g., extract "wi197231" from the product entry)
5. Strip the "wi" prefix to get numeric codes (e.g., 197231 from wi197231)
6. Build the URL: `https://www.ah.nl/mijnlijst/add-multiple?p={code}:{qty}&p={code}:{qty}...`
7. Output the link in TWO formats:
   - First: Clickable markdown link with descriptive text
   - Second: Copyable code block with the raw URL
8. For any unmatched products, output each in its own copyable code block

### JSON Structure
- `supermarkets.json` contains an object with "n": "ah" and "d": [array of products]
- Each product has:
  - "n": product name (e.g., "AH Scharrel kipdijfilet")
  - "l": link with product code (e.g., "wi197231/ah-scharrel-kipdijfilet")
  - "p": price
  - "s": size/quantity description
- Product codes are embedded in the "l" field as "wi{code}/product-slug"
- Extract the numeric part after "wi" to use in the URL

### Search Strategy
- Use Grep with case-insensitive (-i) and regex patterns to find products
- For compound terms, use patterns like `product.*word` to match variations
- Use Bash grep with -B 1 -A 5 flags to get full product context including the "l" field
- The file is large (2.8MB), so use targeted searches rather than reading the whole file
- Run multiple Grep searches in parallel for efficiency (use multiple tool calls in a single message)
- Prefer AH brand products first (e.g., "AH Sesamolie" over "Mee Chun Sesam olie")
- Common product name patterns:
  - "AH" prefix for Albert Heijn house brands
  - "AH Biologisch" for organic products
  - "AH Scharrel" for free-range products
  - Brand names for specialty items (e.g., "Kikkoman", "Verstegen", "Sempio")

### Key Points
- Product codes are numeric, extracted from "wi" links (e.g., 197231 from "wi197231/product-name")
- Quantities follow the product code after a colon (e.g., `197231:1` means 1 unit of product 197231)
- Multiple products are joined with `&` in the URL
- Target audience: Dutch market users
- Match products even if size doesn't exactly match user request (e.g., 248g when user wants 250g)

### Recipe Mode
Recipe mode works in two steps:

**Language Preference:**
- DEFAULT: Provide recipes in Dutch (target audience is Dutch market)
- User can request recipes in English by specifying language preference
- Match the recipe language to the user's request language when possible

**Dietary Restrictions (Default):**
- EXCLUDE fish, oysters, and shrimp from all recipes by default
- ONLY include fish/seafood if the user explicitly requests it

**Step 1: Recipe Selection**
When the user provides a recipe description or request, provide 3-5 recipe options with:
- Recipe title
- Brief description (cuisine type, difficulty)
- Cooking time
- Servings
- Macros (calories, protein, carbs, fat per serving)

**Step 2: Full Recipe**
After the user selects a recipe, provide:

1. **Recipe Title and Description**
   - Name of the dish
   - Brief description (cuisine type, difficulty, cooking time)

2. **Ingredients List**
   - All ingredients with quantities
   - Organized and clearly formatted

3. **Preparation Instructions**
   - Step-by-step cooking instructions
   - Clear, numbered steps
   - Include cooking times and temperatures where relevant

4. **Shopping List Link**
   - Generate the Albert Heijn shopping link for all ingredients
   - Follow the same workflow as grocery list generation
   - Include any products not found in separate code blocks

**Recipe Selection Format:**
```
## Recipe Options

### 1. [Recipe Name]
**Description:** [Brief description]
**Cooking Time:** [Time]
**Servings:** [Number]
**Macros (per serving):** [Calories]kcal | Protein: [X]g | Carbs: [X]g | Fat: [X]g

### 2. [Recipe Name]
...
```

**Full Recipe Output Format:**
```
# [Recipe Name]

**Description:** [Brief description]
**Cooking Time:** [Time]
**Servings:** [Number]

## Ingredients
- [ingredient 1]
- [ingredient 2]
...

## Instructions
1. [Step 1]
2. [Step 2]
...

## Shopping List Link

[Clickable markdown link]

```
[Raw URL in code block]
```

## Products Not Found
[Any unmatched items in separate code blocks]
```
