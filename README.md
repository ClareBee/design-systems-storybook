
### Design Systems
- pure & presentational components only
- no one-off components
- create an inventory: cf Brad Frost, Nathan Curtis
- design tokens: https://medium.com/eightshapes-llc/tokens-in-design-systems-25dd82d58421

- CRA, add components and `component.stories.js` files, stories directory w
- Install Storybook => adds component explorer
```
npx -p @storybook/cli sb init
yarn storybook
```

- Add decorator (wrapper) to apply global style
`.storybook/preview.js`
```javascript
import React from 'react';
import { addDecorator } from '@storybook/react';
import { GlobalStyle } from '../src/shared/global';

addDecorator(story => (
  <>
    <GlobalStyle />
    {story()}
  </>
));
```

- Add link tag for imported styles:
`.storybook/preview-head.html`
e.g.
```html
<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Nunito+Sans:400,700,800,900">
```

### Addons
- register in .storybook/main.js
- **actions addon** -> UI feedback
- **storysource addon** -> see code on the UI:
```
yarn add --dev  @storybook/addon-storysource
```

- **knobs addon** -> dynamic interaction w component props
>knobs don’t replace stories. Knobs are great for exploring the edge cases of the components. Stories are used for showcasing the intended cases.


### Review with Teams
- Design system as single source of truth/single point of failure
- CircleCI for Continuous Integration
### Visual review
>Visual review is the process of confirming the behaviour and aesthetics of user interfaces. It happens both while you’re developing UI and during QA with the team.
From [Storybook](https://www.learnstorybook.com/design-systems-for-developers/react/en/review/)

Deploy for review = ensures shared experience vs divergences on localhost
e.g. via Netlify:
`yarn build-storybook` (build command)
`storybook-static` (publish directory)

PRs run tests and deploy -> copy deploy preview URLs for any changed components/stories and add to your PR description so others can see what's up!

### Test Strategy
- balance comprehensive coverage, straightforward setup, & low maintenance.
>Design systems are composed of UI components. Each UI component includes stories (permutations) that describe the intended look and feel given a set of inputs (props). Stories are then rendered by a browser or device for the end-user.
- from [Storybook](https://www.learnstorybook.com/design-systems-for-developers/react/en/test/)

### Best practices:
- **Articulate supported component states as stories** to clarify which combinations of inputs yields a given state

- **Render components consistently** to mitigate variability

#### Visual Tests
- snapshots - using [Chromatic](https://docs.chromaticqa.com/) (visual testing tool for Storybook, captures a baseline image of every story)
`yarn add --dev storybook-chromatic`
`yarn chromatic test --app-code=<app-code>`

Add to config.yml for CircleCI:
```yml
 - run: yarn chromatic test --app-code=<app-code> --exit-zero-on-changes`
 ```
Add script:
```json
{
  "scripts": {
    "chromatic": "CHROMATIC_APP_CODE=<app-code> chromatic"
  }
}
```

#### Unit Tests
>Unit tests verify whether the UI code returns the correct output given a controlled input. They live alongside the component and help you validate specific functionality.

- avoid too many - can be cumbersome

#### Accessibility Tests
c.15% of the population has disabilities
`yarn add --dev @storybook/addon-a11y` & register in `addons.js`
```javascript
import '@storybook/addon-a11y/register';
```
- add to `config.js`:
```javascript
import { withA11y } from '@storybook/addon-a11y';
addDecorator(withA11y);
```

### Document for Stakeholders
Ideal world:
- up-to-date
- easy tools e.g. markdown
- easy to maintain e.g. with boilerplate
- customisable where needed
From [Storybook](https://www.learnstorybook.com/design-systems-for-developers/react/en/document/)

#### Storybook Docs & JSDoc
`yarn add --dev @storybook/addon-docs` & add to `addons.js`
e.g.
```javascript
/**
- Use an avatar for attributing actions or content to specific users.
- The user's name should always be present when using Avatar – either printed beside the avatar or in a tooltip.
**/
// also:
sizes.story = {
  parameters: { docs: { storyDescription: '4 sizes are supported.' } },
};
```

#### MDX
[MDX](https://mdxjs.com/)

- lets you use JSX inside Markdown
Enable in `config.js`:
```javascript
// automatically import all files ending in *.stories.js|mdx
configure(
  [
    require.context('../src', false, /Intro\.stories\.mdx/),
    require.context('../src', true, /\.stories\.(js|mdx)$/),
  ],
  module
);
```

- Storybook uses built-in Doc Blocks (interactive previews, props tables etc.)
- keep defaults by wrapping in a Preview
```javascript
import { Meta, Story, Props, Preview } from '@storybook/addon-docs/blocks';

# …

<Preview withToolbar>
  <Story name="standard">
    <Avatar
      size="large"
      username="Tom Coleman"
      src="https://avatars2.githubusercontent.com/u/132554"
    />
  </Story>
</Preview>

<Props of={Avatar} />
```

#### Doc-only mdx pages
e.g. `src/Intro.stories.mdx`
```javascript
import { Meta } from '@storybook/addon-docs/blocks';

<Meta title="Design System|Introduction" />

# Introduction to the Learn Storybook design system

The Learn Storybook design system is a subset of the full [Storybook design system](https://github.com/storybookjs/design-system/), created as a learning resource for those interested in learning how to write and publish a design system using best practice techniques.

Learn more at [Learn Storybook](https://learnstorybook.com).
```
Load in first in `config.js`
```javascript
configure(
  [
    require.context('../src', false, /Intro\.stories\.mdx/),
    require.context('../src', true, /\.stories\.(js|mdx)$/),
  ],
  module
);
```
