
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

### Distribute
- Packaging (Common UI components, design tokens, docs)
e.g. via npm
1. Add README.md
2. Add index.js as common entry point & also export design tokens/components
3. Add dev dependency for Babel to compile JS: `yarn add --dev @babel/cli`
4. Add build scripts:
```json
{
  "scripts": {
    "build": "BABEL_ENV=production babel src -d dist",
      ...
  }
  "babel": {
    "presets": [
      "react-app"
    ]
  }
}
```
5. Update `package.json` to ensure all info
e.g. w `yarn init --a` (initialises the package for publication)

#### Release management with Auto
Auto: https://intuit.github.io/auto/pages/getting-started.html
>Automated releases powered by pull request labels. Streamline your release workflow and publish constantly!

`yarn add --dev auto`

In your `.env`:
```
GH_TOKEN=<value from GitHub>
NPM_TOKEN=<value from npm> (https://www.npmjs.com/settings/%3Cyour-username%3E/tokens)
```
- Use Auto to create labels on GitHub:
`yarn auto create-labels`
- Tag all future PRs w one of the labels: major, minor, patch, skip-release, prerelease, internal, documentation before merging
- `yarn auto changelog` - w every commit so far
```
git add CHANGELOG.md
git commit -m "Changelog for v0.1.0 [skip ci]"
npm version 0.1.0 -m "Bump version to: %s [skip ci]"
npm publish
```
- NB: [skip ci] in your commit message to skip the CircleCI build
- Use Auto to create a release on GitHub:
```
git push --follow-tags origin master
yarn auto release
```
- Set up scripts to use Auto:
```json
{
  "scripts": {
    "release": "auto shipit"
  }
}
```
- Add to CircleCI:
```yml
# ...
- run: yarn test
- run: yarn chromatic test -a 2wix88i1ziu
- run: |
    if [ $CIRCLE_BRANCH = "master" ]
    then
      yarn release
    fi
```
- Add env variables to CircleCI: https://circleci.com/gh/%3Cyour-username%3E/learnstorybook-design-system/edit#env-vars
---
### Using the Design System in an App
- run app's storybook:
```
yarn install
yarn storybook
```
- add ours as a dependency:
```
yarn add <your-username>-learnstorybook-design-system
```
- add to config.js
- add global decorator to .storybook/preview.js:
```javascript
import React from 'react';
import { addDecorator } from '@storybook/react';
import { global as designSystemGlobal } from '<your-username>-learnstorybook-design-system';

const { GlobalStyle } = designSystemGlobal;

addDecorator(story => (
  <>
    <GlobalStyle />
    {story()}
  </>
));
```
>Showcasing the design system during feature development increases the likelihood that developers will reuse existing components instead of wasting time inventing their own.
