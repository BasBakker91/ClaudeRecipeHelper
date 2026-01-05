# Albert Heijn Link Generator

## Project Overview

This project generates Albert Heijn shopping list links from a list of groceries. It takes grocery items as input, matches them against the Albert Heijn product database, and creates shareable shopping list URLs.

## Project Structure

```
AlbertHeijnLinkGenerator/
├── .claude/                 # Claude Code configuration
│   └── settings.local.json  # Local settings for Claude Code
├── supermarkets.json        # Albert Heijn product database (16,173 products)
├── favorites.json           # User's favorite recipes
├── common-products.json     # Frequently used products cache (for faster lookups)
└── CLAUDE.MD                # This file - project documentation for Claude
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
- Save shopping lists and recipes to files for future reference
- Manage favorite recipes and include them in recipe suggestions

## Updating Product Database

The Albert Heijn product database should be updated periodically to ensure accurate product matching and pricing.

**Source:** https://raw.githubusercontent.com/supermarkt/checkjebon/refs/heads/main/data/supermarkets.json

**Update Process:**
1. Download the latest data: `curl -o supermarkets_temp.json "https://raw.githubusercontent.com/supermarkt/checkjebon/refs/heads/main/data/supermarkets.json"`
2. Filter for Albert Heijn only and save: `python -c "import json; data = json.load(open('supermarkets_temp.json', 'r', encoding='utf-8')); ah = next((s for s in data if s['n'] == 'ah'), None); json.dump(ah, open('supermarkets.json', 'w', encoding='utf-8'), ensure_ascii=False, indent=2)"`
3. Verify the update: `python -c "import json; data = json.load(open('supermarkets.json', 'r', encoding='utf-8')); print('Supermarket:', data['n']); print('Total products:', len(data['d']))"`
4. Clean up: `rm supermarkets_temp.json` (Windows: `del supermarkets_temp.json`)

**Current Database Stats:**
- Last updated: 2026-01-04
- Total Albert Heijn products: 16,173
- File size: ~2.4 MB

## Technology Stack

- Python 3.x (for data processing and filtering)
- Bash/Command line tools (curl for downloads)
- JSON for data storage

## Notes for Claude

### Workflow
1. User provides a list of grocery items (with quantities)
2. **Check Common Products Cache first:** Look up items in `common-products.json` for instant matches
3. For items NOT in cache: Run parallel Grep searches for all products at once using case-insensitive regex patterns
4. From Grep results, identify the best match for each item (prefer AH brand, match size if specified)
5. Extract product codes from the "l" field using Bash grep (e.g., extract "wi197231" from the product entry)
6. Strip the "wi" prefix to get numeric codes (e.g., 197231 from wi197231)
7. Build the URL: `https://www.ah.nl/mijnlijst/add-multiple?p={code}:{qty}&p={code}:{qty}...`
8. Output the link in TWO formats:
   - First: Clickable markdown link with descriptive text
   - Second: Copyable code block with the raw URL
9. For any unmatched products, output each in its own copyable code block

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

### Common Products Cache

**Purpose:**
Speed up shopping list generation by maintaining a cache of frequently used products, avoiding repeated searches through the large supermarkets.json file.

**File:** `common-products.json`

**Structure:**
```json
{
  "products": [
    {
      "name": "eieren",
      "aliases": ["eggs", "scharreleieren", "vrije uitloop eieren"],
      "productCode": "588275",
      "fullName": "AH Scharreleieren",
      "category": "proteins-dairy",
      "notes": "6 stuks pack"
    },
    {
      "name": "kipfilet",
      "aliases": ["chicken breast", "kip", "scharrel kipfilet"],
      "productCode": "598811",
      "fullName": "AH Biologisch Scharrel kipfilet",
      "category": "proteins-dairy",
      "notes": ""
    }
  ],
  "metadata": {
    "lastUpdated": "2026-01-05",
    "totalProducts": 2
  }
}
```

**Usage Instructions:**

1. **Lookup Priority:**
   - ALWAYS check `common-products.json` FIRST before searching supermarkets.json
   - Match against both "name" and "aliases" fields (case-insensitive)
   - If found in cache, use the product code immediately
   - Only search supermarkets.json for products NOT in cache

2. **Adding Products:**
   - User can say "add [product] to common products" or "cache this product"
   - Automatically add products that are used in multiple recipes or meal plans
   - Include relevant aliases (Dutch and English terms)
   - Categorize: "proteins-dairy", "fruits", "vegetables", "pantry", "other"

3. **Removing Products:**
   - User can say "remove [product] from common products"
   - Remove by name or product code

4. **Updating Products:**
   - User can say "update [product] in common products"
   - Useful when product codes change or better alternatives are found

5. **Viewing Cache:**
   - User can say "show common products" or "list cached products"
   - Display organized by category

**Benefits:**
- Faster shopping list generation (no need to search 16k+ products)
- Consistent product selection (always picks the same preferred brand)
- Reduces API/file read overhead
- User can customize their preferred products

**Maintenance:**
- Update product codes when AH changes them
- Add new frequently-used products as they come up
- Remove products that are no longer available or rarely used

### Recipe Mode
Recipe mode works in two steps:

**Language Preference:**
- DEFAULT: Provide recipes in Dutch (target audience is Dutch market)
- User can request recipes in English by specifying language preference
- Match the recipe language to the user's request language when possible

**Personalized Recipe Modes:**

The system supports two distinct recipe modes based on who the recipe is for:

**Mode 1: "For Me" (Individual - High Protein Carnivore + Fruit)**
- Target: 40g+ protein per serving
- Allowed foods: ONLY animal products (meat, fish, seafood, eggs, dairy) and fruit
- Excluded foods: ALL grains, legumes, vegetables
- Fish and seafood: INCLUDED (allowed in this mode)
- Default language: Dutch
- Header display: `🥩 Recipe Mode: For Me (High Protein Carnivore + Fruit)`

**Mode 2: "For Us" (Shared - Standard)**
- Standard recipe parameters (no specific macros target)
- Dietary restrictions: EXCLUDE fish, oysters, and shrimp by default
- Fish/seafood: ONLY include if user explicitly requests it
- Default language: Dutch
- Header display: `👥 Recipe Mode: For Us (Standard)`

**Mode Selection:**
- User will specify "for me" or "for us" in their recipe request
- If unclear which mode to use, ALWAYS ask the user to clarify before suggesting recipes
- Display the appropriate mode header when outputting recipe suggestions or full recipes

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

### Favorites System

**Data Storage:**
- Favorite recipes are stored in `favorites.json`
- Each recipe includes:
  - Recipe name, description, cuisine type, difficulty
  - Cooking time and servings
  - Macros (calories, protein, carbs, fat per serving)
  - Ingredients with quantities and optional notes
  - Step-by-step instructions
  - Tips and related recipes
  - Tags for categorization and searching (ALWAYS include bilingual tags - both Dutch and English)
  - Date added and optional Albert Heijn shopping list URL

**Tag Strategy:**
- ALWAYS use bilingual tags (both Dutch and English) for maximum searchability
- Include both languages for key descriptors:
  - Cuisine: "koreaans" + "korean", "italiaans" + "italian", "aziatisch" + "asian"
  - Characteristics: "spicy" + "pittig", "quick" + "snel", "easy" + "makkelijk"
  - Protein/Nutrients: "high protein" + "hoog eiwit", "low carb" + "koolhydraatarm"
  - Meal type: "weeknight dinner" + "doordeweeks", "meal prep" + "maaltijdvoorbereiding"
- This allows users to request recipes in either Dutch or English and get matching results

**Adding to Favorites:**
- User can say "add this to favorites" or "save this recipe" after viewing a full recipe
- Claude will add the recipe to favorites.json with a unique ID
- Update metadata (lastUpdated, totalRecipes count)

**Using Favorites in Recipe Suggestions:**
- When user asks for recipe suggestions, ALWAYS search favorites first
- Match based on:
  - Similar tags (e.g., "noodle bowl", "ramen", "Asian", "quick")
  - Related cuisine types
  - Cooking time preferences
  - Dietary preferences
- Present favorites FIRST in the suggestion list, clearly marked with a star indicator
- Then show new recipe suggestions

**Favorites Display Format:**
```
## Recipe Options

### From Your Favorites ⭐

### 1. [Favorite Recipe Name]
**Description:** [Brief description]
**Cooking Time:** [Time]
**Servings:** [Number]
**Macros (per serving):** [Calories]kcal | Protein: [X]g | Carbs: [X]g | Fat: [X]g

### New Suggestions

### 2. [New Recipe 1]
...
```

**Saving Recipes and Shopping Lists to Files:**
- When user requests to save a recipe or shopping list, save it as a markdown (.md) file
- Use descriptive filenames (e.g., `korean-chicken-noodle-bowl.md`, `weekly-groceries.md`)
- Include all recipe details: ingredients, instructions, macros, shopping links
- For shopping lists: include item list, quantities, generated AH link, and products not found
- Files can be reused later to regenerate shopping links or review recipes

### Meal Planning Feature

**Overview:**
The meal planning feature helps users plan meals for multiple days, automatically consolidating shopping lists and managing both "For Me" and "For Us" recipes in a single plan.

**Planning Modes:**

1. **Mixed Mode (Default):**
   - Combine "For Me" and "For Us" recipes in one plan
   - Specify which meals are individual vs. shared
   - Example: "Plan 5 days - breakfast and lunch for me, dinner for us"

2. **Single Mode:**
   - All recipes follow one mode (either "For Me" or "For Us")
   - Example: "Plan 7 days of meals for me"

**Meal Plan Structure:**

When user requests a meal plan, ask for:
- Duration (number of days: 3, 5, 7, 14, etc.)
- Meals per day (breakfast, lunch, dinner, snacks)
- Which meals are "for me" vs "for us"
- Any preferences (cuisine types, cooking time limits, ingredient preferences)
- Whether to include favorites or only new recipes

**Meal Plan Output Format:**

```
# [Duration] Meal Plan

**Mode:** [Mixed / For Me / For Us]
**Duration:** [X days]
**Total Recipes:** [X]

## Daily Breakdown

### Day 1 - [Date]
**Breakfast** 🥩 For Me
- [Recipe name] | [Cook time] | [Macros]

**Lunch** 🥩 For Me
- [Recipe name] | [Cook time] | [Macros]

**Dinner** 👥 For Us
- [Recipe name] | [Cook time] | [Macros]

### Day 2 - [Date]
...

## Weekly Macro Summary (For Me meals only)
- Total Calories: [X]kcal
- Average Daily Protein: [X]g
- Average Daily Carbs: [X]g
- Average Daily Fat: [X]g

## Consolidated Shopping List

**Proteins & Dairy:**
- [ingredient]: [total quantity needed] (used in: Day 1 Breakfast, Day 3 Dinner)
- [ingredient]: [total quantity]

**Fruits:**
- [ingredient]: [total quantity]

**Other:**
- [ingredient]: [total quantity]

## Albert Heijn Shopping Link

[Click here to add all ingredients to your AH shopping list](url)

```
url
```

## Products Not Found
[Any unmatched items]

## Meal Prep Tips
- [Ingredient] can be prepped on [Day] for use in [X recipes]
- [Recipe] from Day X makes great leftovers for Day Y lunch
- Batch cook [ingredient] for Day X and Day Y to save time
```

**Smart Features:**

1. **Ingredient Optimization:**
   - Group recipes that share ingredients to minimize waste
   - Suggest bulk cooking when same ingredient appears in multiple recipes
   - Identify leftover opportunities (e.g., roasted chicken on Day 1 becomes chicken salad on Day 2)

2. **Shopping List Consolidation:**
   - Combine all ingredients across all recipes
   - Sum quantities (e.g., "500g chicken" + "300g chicken" = "800g chicken")
   - Show which recipes use each ingredient
   - Generate single AH shopping list for entire plan

3. **Variety & Balance:**
   - Avoid repeating same protein source on consecutive days
   - Mix cuisine types throughout the week
   - Balance cooking times (quick meals on busy days, longer recipes on weekends)
   - Consider "For Me" macro targets across the week (40g+ protein per meal)

4. **Leftover Management:**
   - Identify recipes that scale well for multiple servings
   - Suggest leftover-based meals (e.g., "Day 1 dinner serves 4, use leftovers for Day 2 lunch")
   - Mark recipes as "great for meal prep"

**Storage:**
- Save meal plans to `meal-plans/` directory
- Filename format: `meal-plan-YYYY-MM-DD-to-YYYY-MM-DD.md`
- Include all recipe details, shopping list, and macro summaries
- Can be referenced later to regenerate shopping links or repeat successful plans

**User Requests Examples:**
- "Plan 5 days of meals for me"
- "Create a week of dinners for us"
- "Plan 7 days - breakfast and lunch for me, dinner for us"
- "Make a 3-day meal plan with only quick recipes (under 30 min)"
- "Plan next week's meals using my favorites"
