# Styleguide

## Language

### Headings and subheadings

- Be concise; headings and subheadings should be scannable and use only as many words are needed to convey their meaning
- Write in sentence case since it feel less stiff and formal
- Don't use punctuation like periods or semicolons

> ✅ Get started with your proposal

> ❌ Get Started with Your Proposal.

#### Microcopy

For labels and microcopy, prioritize short and actionable content by removing articles like "the", 'this", and "an".

> ✅ Download now

> ❌ Download the proposal PDF

### Buttons

Always write button labels in sentence case.

> ✅ Get started

> ❌ Get Started

#### Buttons for action

When buttons enable user actions, the formula is `verb` + `noun`.

> ✅ Edit decor

> ❌ Edit

### Links

Links should contain text like "click here" or "learn more", but should be self-sufficient, i.e., if you read nothing but the link text, you should know where it takes you.

> ✅ Find out [more about Carlo](https://carlobadini.com)

> ❌ To find out more about Carlo, [click here](https://carlobadini.com)

#### Links to webpages

When adding links to other pages, you should ideally keep the title of the page somewhere in context. For example, when linking to an article on a website, link to the article title:

> ✅ To learn how to edit files, see [Editing files in your repository](https://docs.github.com/en/free-pro-team@latest/github/managing-files-in-a-repository/editing-files-in-your-repository) on the GitHub website

> ❌ Learn [how to edit files](https://docs.github.com/en/free-pro-team@latest/github/managing-files-in-a-repository/editing-files-in-your-repository)

## Style

### Capitalization

Use sentence case everywhere, including:

- Headings and subheadings
- UI copy
- Buttons
- Links
- Error messages
- Confirmation messages
- Success messages
- Placeholders
- Input labels

Render proper nouns as their creators/trademarks prefer: GitHub, npm, Facebook.

### Numbers

Use the numeral when numbers appear in copy. This includes ordinals:

> ✅ 30+ results found

> ❌ More than 30 results found

When writing copy for the website, use the most common Swiss locale for numbers:

> ✅ 1'823'231

This can be done in JavaScript using `1000.toLocaleString("de-ch")`

When writing for an international audience (such as a GitHub project README), use US English and number locale.

#### Dates

When date is written in numerals, the standard format is YYYY-MM-DD.

> ✅ 2020-05-02

> ❌ 05-02-2020

#### Ranges

Use an en dash (–) to indicate a range or span of numbers. Do not use spaces before and after the en dash.

> ✅ Viewing items 100–150

#### Money

In most cases, Swiss Francs (CHF) are used. Ensure that the "CHF" is before the number and separated by a space.

> ✅ CHF 1'000

> ❌ 1000CHF

When using a currency symbol such as $, no space is required.

> ✅ $99

When writing about monthly pricing, follow the convention: $0/mo. Do not use spaces. In expanded copy, you can use the phrase "per month".

> ✅ CHF 225/month

> ✅ $225 per month

> ❌ 225 CHF/month

#### Time

When writing about a time of the day, use 24-hour time. When writing 12-hour times, add a space between lowercase "am" or "pm".

> ✅ 23:00

> ✅ 11 pm

> ❌ 11pm

#### Commas

Prefer using serial commas when writing lists.

#### Periods

Use periods in complete sentences. Don't use periods in lists.

### Writing about Koj

Make sure you don't change the capitalization of the world.

> ✅ Koj

> ❌ KOJ

###

## Credits

Parts of this styleguide are based on [Sourcegraph content guidelines](https://about.sourcegraph.com/handbook/communication/content_guidelines), licensed MIT.
