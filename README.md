
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
