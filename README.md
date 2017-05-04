# Tad Sellers' Blog

A [Jekyll](https://jekyllrb.com/) blog built with [Jekyll Now](https://github.com/barryclark/jekyll-now) as a base.

## Technical Notes

Cause what else am I gonna do with the readme file of a Github Pages website?

### Deep Anchor Links

I've re-created the feature of markdown files on GitHub where hovering over a heading produces a visible, clickable anchor link. These are called deep anchor links. They're really helpful for visitors wanting to link to a specific part of a longer page.

I originally wanted to create a Jekyll plugin for this, but [custom plugins don't work](https://jekyllrb.com/docs/plugins/) on blogs hosted on Github Pages. The workaround is to have Javascript add the links. I used [AnchorJS](https://www.bryanbraun.com/anchorjs/#icon-font) for this.