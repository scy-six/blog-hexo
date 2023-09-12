---
title: tags
tags:

  - testing
category:
  - 未分类
password:
message:
description: 未添加描述
date: 2023-09-07 00:43:18
cover:
---
# button.js

/**
 * button.js | https://theme-next.js.org/docs/tag-plugins/button
 */

'use strict';

module.exports = ctx => function(args) {
  args = args.join(' ').split(',');
  const url   = args[0];
  const text  = (args[1] || '').trim();
  let icon    = (args[2] || '').trim();
  const title = (args[3] || '').trim();

  if (!url) {
    ctx.log.warn('URL can NOT be empty.');
  }
  if (icon.length > 0) {
    if (!icon.startsWith('fa')) icon = 'fa fa-' + icon;
    icon = `<i class="${icon}"></i>`;
  }

  return `<a class="btn" href="${url}"${title.length > 0 ? ` title="${title}"` : ''}>${icon}${text}</a>`;
};
# caniuse.js
/**
 * caniuse.js | https://theme-next.js.org/docs/tag-plugins/caniuse
 */

'use strict';

module.exports = ctx => function(args) {
  const [feature, periods = 'current'] = args.join('').split('@');

  if (!feature) {
    ctx.log.warn('Caniuse feature can NOT be empty.');
    return '';
  }

  return `<iframe data-feature="${feature}" src="https://caniuse.bitsofco.de/embed/index.html?feat=${feature}&periods=${periods}&accessible-colours=false" frameborder="0" width="100%" height="400px"></iframe>`;
};
# center-quote.js
/**
 * center-quote.js | https://theme-next.js.org/docs/tag-plugins/
 */

'use strict';

module.exports = ctx => function(args, content) {
  return `<blockquote class="blockquote-center">
${ctx.render.renderSync({ text: content, engine: 'markdown' })}
</blockquote>`;
};
# index.js
/* global hexo */

'use strict';

const postButton = require('./button')(hexo);

hexo.extend.tag.register('button', postButton);
hexo.extend.tag.register('btn', postButton);

const caniUse = require('./caniuse')(hexo);

hexo.extend.tag.register('caniuse', caniUse);
hexo.extend.tag.register('can', caniUse);

const centerQuote = require('./center-quote')(hexo);

hexo.extend.tag.register('centerquote', centerQuote, true);
hexo.extend.tag.register('cq', centerQuote, true);

const groupPicture = require('./group-pictures')(hexo);

hexo.extend.tag.register('grouppicture', groupPicture, true);
hexo.extend.tag.register('gp', groupPicture, true);

const postLabel = require('./label')(hexo);

hexo.extend.tag.register('label', postLabel);

const linkGrid = require('./link-grid');

hexo.extend.tag.register('linkgrid', linkGrid, true);
hexo.extend.tag.register('lg', linkGrid, true);

const mermaid = require('./mermaid');

hexo.extend.tag.register('mermaid', mermaid, true);

const wavedrom = require('./wavedrom');

hexo.extend.tag.register('wavedrom', wavedrom, true);

const postNote = require('./note')(hexo);

hexo.extend.tag.register('note', postNote, true);
hexo.extend.tag.register('subnote', postNote, true);

const pdf = require('./pdf')(hexo);

hexo.extend.tag.register('pdf', pdf);

const postTabs = require('./tabs')(hexo);

hexo.extend.tag.register('tabs', postTabs, true);
hexo.extend.tag.register('subtabs', postTabs, true);
hexo.extend.tag.register('subsubtabs', postTabs, true);

const postVideo = require('./video');

hexo.extend.tag.register('video', postVideo);
# label.js
/**
 * label.js | https://theme-next.js.org/docs/tag-plugins/label
 */

'use strict';

module.exports = ctx => function(args) {
  const [classes = 'default', text = ''] = args.join(' ').split('@');

  if (!text) ctx.log.warn('Label text must be defined!');

  return `<mark class="label ${classes.trim()}">${text}</mark>`;
};
# link-grid.js
/**
 * link-grid.js | https://theme-next.js.org/docs/tag-plugins/link-grid
 */

'use strict';

module.exports = function([image = '/images/avatar.gif', delimiter = '|', comment = '%'], content) {
  const links = content.split('\n').filter(line => line.trim() !== '').map(line => {
    const item = line.split(delimiter).map(arg => arg.trim());
    if (item[0][0] === comment) return '';
    const imageSource = item[3] || image;
    const hasExtension = /\.[^/]+$/.test(imageSource);
    return `<div class="link-grid-container">
<object class="link-grid-image" ${hasExtension ? '' : 'type="image/jpeg" '}data="${imageSource}"></object>
<p>${item[0]}</p><p>${item[2] || item[1]}</p>
<a href="${item[1]}"></a>
</div>`;
  });
  return `<div class="link-grid">${links.join('')}</div>`;
};
# mermaid.js
/**
 * mermaid.js | https://theme-next.js.org/docs/tag-plugins/mermaid
 */

'use strict';

const { escapeHTML } = require('hexo-util');

module.exports = function(args, content) {
  return `<pre class="mermaid">
${args.join(' ')}
${escapeHTML(content)}
</pre>`;
};
# note.js
/**
 * note.js | https://theme-next.js.org/docs/tag-plugins/note
 */

'use strict';

module.exports = ctx => function(args, content) {
  const keywords = ['default', 'primary', 'info', 'success', 'warning', 'danger', 'no-icon'];
  const className = [];
  for (let i = 0; i < 2; i++) {
    if (keywords.includes(args[0])) {
      className.push(args.shift());
    } else {
      break;
    }
  }

  content = ctx.render.renderSync({ text: content, engine: 'markdown' });
  if (args.length === 0) {
    return `<div class="note ${className.join(' ')}">${content}</div>`;
  }
  return `<details class="note ${className.join(' ')}"><summary>${ctx.render.renderSync({ text: args.join(' '), engine: 'markdown' })}</summary>
${content}
</details>`;
};
# pdf.js
/**
 * pdf.js | https://theme-next.js.org/docs/tag-plugins/pdf
 */

'use strict';

module.exports = ctx => function(args) {
  const theme = ctx.theme.config;
  return `<div class="pdf-container" data-target="${args[0]}" data-height="${args[1] || theme.pdf.height}"></div>`;
};
# tabs.js
/**
 * tabs.js | https://theme-next.js.org/docs/tag-plugins/tabs
 */

'use strict';

module.exports = ctx => function(args, content = '') {
  const tabBlock = /<!--\s*tab (.*?)\s*-->\n([\w\W\s\S]*?)<!--\s*endtab\s*-->/g;

  args = args.join(' ').split(',');
  const tabName = args[0];
  const tabActive = Number(args[1]) || 0;

  let tabId = 0;
  let tabNav = '';
  let tabContent = '';

  if (!tabName) ctx.log.warn('Tabs block must have unique name!');
  const matches = content.matchAll(tabBlock);

  for (const match of matches) {
    let [caption = '', icon = ''] = match[1].split('@');
    let postContent = match[2];

    postContent = ctx.render.renderSync({ text: postContent, engine: 'markdown' }).trim();

    const abbr = tabName + ' ' + ++tabId;
    const href = abbr.toLowerCase().split(' ').join('-');

    icon = icon.trim();
    if (icon.length > 0) {
      if (!icon.startsWith('fa')) icon = 'fa fa-' + icon;
      icon = `<i class="${icon}"></i>`;
    }

    caption = icon + caption.trim();

    const isActive = (tabActive > 0 && tabActive === tabId) || (tabActive === 0 && tabId === 1) ? ' active' : '';
    tabNav += `<li class="tab${isActive}"><a href="#${href}">${caption || abbr}</a></li>`;
    tabContent += `<div class="tab-pane${isActive}" id="${href}">${postContent}</div>`;
  }

  tabNav = `<ul class="nav-tabs">${tabNav}</ul>`;
  tabContent = `<div class="tab-content">${tabContent}</div>`;

  return `<div class="tabs" id="${tabName.toLowerCase().split(' ').join('-')}">${tabNav + tabContent}</div>`;
};
# video.js
/**
 * video.js | https://theme-next.js.org/docs/tag-plugins/
 */

'use strict';

module.exports = function(args) {
  return `<video src="${args[0]}" preload="metadata" controlslist="nodownload" controls playsinline poster="${args[1] || ''}"></video>`;
};
# wavedrom.js
/**
 * wavedrom.js | https://theme-next.js.org/docs/tag-plugins/wavedrom
 */

'use strict';

module.exports = function(args, content) {
  return `<div class="wavedrom"><script type="WaveDrom">
${content}
</script></div>`;
};