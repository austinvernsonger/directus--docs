# Custom UIs Introduction

## UI Structure

For most content types the Directus Core User-Interfaces should work well. However if you're looking for a specific interface or have proprietary or custom content types to manage – you may need or want to create something more tailored. Directus UIs are modular Javascript files (or dedicated sub-directories for complex interfaces) that allow you to create new ways of interfacing with database content.

Text inputs, checkboxes, and WYSIWYG editors are pretty standard – but perhaps your application needs a JSON editor, unique hashtag generator, iTunes song lookup, or custom seating-chart. So let's look at how these would be created within Directus, starting with the components of the UI.

#### ID

Each UI must have a unique identifier. For instance the Single File ID is `single_file`.

#### Datatypes

Each UI works with a specific set of datatypes, for instance a _Text Area_ UI might work with `TEXT` or `VARCHAR` datatypes, while a _Single File_ UI only works with `INT` (storing the related file ID).

#### Options

Options are the key to making the UI usable in different circumstances. Options are easy to create, and actually use OTHER UIs as the interface for setting the value (so meta). Below is an example of an dropdown option to determine the size of an interface:

```
{id: 'size', ui: 'select', required: true, options: {options: {'large':'Large','medium':'Medium','small':'Small'} }, comment: "Choose a size!"},
```

#### Template

Whether it's simple enough to add (escaped) in the single Javascript file, or to be included from a separate template file – this is where you define your markup and any additional styling. Directus uses the Handlebars templating library – so be sure to reference their Docs for any questions you may have.

#### Listing

In addition to how the content is manipulated on the Edit Item page, you also need to consider how the data will be presented on the Item Listing page (more tabular). Each UI has a separate `list` function to handle that view.

#### Validation

Of course you're going to want to ensure that data is saved correctly into the column – so you should always include logic in the `validate` function.

## Creating Your First UI

The easiest way to get started is to take one of the existing Core UIs and duplicate it. If possible, try to find one with similar functionality that supports the same datatype(s). Give it a unique name and save it into the `/customs/ui` directory – at this point if you refresh Directus and create a new field you will see your new UI as an option! Now you can just edit as needed the logic, template, validation, listing format, and event handlers!

