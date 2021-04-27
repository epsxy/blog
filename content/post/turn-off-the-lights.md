---
title: "Turn off the lights ✨"
date: 2020-09-08T16:00:56+02:00
draft: false
tags: []
categories: ["dev"]
---

*On adding a dark/light theme to hugo*

## Introduction

In the past few years, almost every software and blog started to add the possibility to switch between a light and a dark mode. I am myself a huge enthusiast of it and turned on dark mode on almost all the apps and websites I use regularly. However, until now, my personal blog did not had this feature yet. What a shame. This article relates my journey of adding this feature.

## Theming hugo

The principle of theming hugo is adding your desired theme git repository as a submodule under `./blog/themes/` and then, to declare the theme name in `./blog/config.toml`:

```
theme = "${THEME_NAME}"
```

I am currently using [Hikari](https://github.com/digitalcraftsman/hugo-hikari-theme) theme, by [digitalcraftsman](https://github.com/digitalcraftsman). As far as I know, this theme does not include the possibility to switch between themes yet and it's also archived and not maintained anymore. So after forking it, I decided to update it for my own usage.

## Theming CSS

Althought I don't have a lot of experience regarding theming web applications in CSS, I think the most easy way is to add a specific class to the `<body>` HTML tag depending on the theme we want to apply to the page. For example `<body class="dark"></body>` for a dark theme or `<body class="light"></body>` for a light theme.

Then, we have to write specific CSS targeting all elements we want to style under the `.dark` (or `.light` or `.${THEME}`) class. For example, `.dark header h1 { color: #fff }`.

## Switching between themes

Considering what we said in the previous paragraph, we want to add buttons to dynamically add and remove classes on the `<body>` HTML tag.

### Icons for light and dark theme buttons

[Material Icons](https://material.io/resources/icons) are beautiful and ready to be used! Sun and moon icons will be perfect for choosing between light and dark theme.

![Drawning of pink moutains](/img/articles/turn-off-the-lights/symbols.png)

### Changing class on `<body>` tag dynamically

#### Principles

This behavior relies on the following steps:

- Adding 2 buttons, one for the light theme and one for the dark theme
- Choosing a default theme, which means hiding a button by default (hiding the light theme button if the default theme is light, or hiding the dark theme button if the default theme is dark)
- Setting up click handlers on the buttons, so that onclick, we hide the button, display the opposite button, and change the class on `<body>` Accordingly. For example, if we click on the *dark theme* button, we have to hide the dark theme button, display the light theme button and add the `dark` class to `<body>`

At first, I have implemented this feature in the main javascript file of the hugo theme, loaded in the callback `window.onload`. However, I met some latency issues (where the light theme was displayed during a second and then the black theme were applied. This was not really satisfying because the interface was *blinking*) and I decided to inline the javascript file of this theming behavior directly after the html template.


#### Implementation

```html
<div>
    <button class="sun-btn" title="Light mode">
        <svg/><!--INSERT ICON SVG--></svg>
    </button>
    <button class="moon-btn" title="Dark mode">
        <svg/><!--INSERT ICON SVG--></svg>
    </button>
</div>
<script src="{{ .Site.BaseURL }}js/header.js"></script>
```

```javascript
const $sunIcon = document.getElementsByClassName('sun-btn')[0],
    $moonIcon = document.getElementsByClassName('moon-btn')[0];

const applyDarkTheme = () => {
    document.body.classList.add('dark');
    document.body.classList.remove('light');
    removeClass($sunIcon, 'hidden');
    addClass($moonIcon, 'hidden');
};

const applyLightTheme = () => {
    document.body.classList.add('light');
    document.body.classList.remove('dark');
    addClass($sunIcon, 'hidden');
    removeClass($moonIcon, 'hidden');
};

/* apply default theme */
applyDarkTheme();

$sunIcon.addEventListener('click', applyLightTheme, false);
$moonIcon.addEventListener('click', applyDarkTheme, false);

function addClass(element, className) {
    element.className += ' ' + className;
}

function removeClass(element, className) {
    const classNameRegEx = new RegExp('\\s*' + className + '\\s*');
    element.className = element.className.replace(classNameRegEx, ' ');
}
```

### Saving selected theme

#### Principles

To store the user configuration, we can use the browser [LocalStorage](https://developer.mozilla.org/fr/docs/Web/API/Window/localStorage) API, which allows us to store key-value properties. So we can store the theme information (`light|dark`) under the `theme` key. The API is very easy to use: 

```javascript
// set value
localStorage.setItem('theme', 'dark');
// get value
const theme = localStorage.getItem('theme');
// remove value
localStorage.removeItem('theme');
```

We have to add the `LocalStorage` calls inside the previous javascript snippet, i.e.:

- Save the `dark` value in `LocalStorage` inside `applyDarkTheme` function
- Save the `light` value in `LocalStorage` inside the `applyLightTheme` function
- Read `LocalStorage` on page load and apply the theme accordingly.

#### Implementation

```javascript
const $sunIcon = document.getElementsByClassName('sun-btn')[0],
    $moonIcon = document.getElementsByClassName('moon-btn')[0];

const applyDarkTheme = () => {
    /* store dark theme in LocalStorage */
    localStorage.setItem('theme', 'dark');
    document.body.classList.add('dark');
    document.body.classList.remove('light');
    removeClass($sunIcon, 'hidden');
    addClass($moonIcon, 'hidden');
};

const applyLightTheme = () => {
    /* store light theme in LocalStorage */
    localStorage.setItem('theme', 'light');
    document.body.classList.add('light');
    document.body.classList.remove('dark');
    addClass($sunIcon, 'hidden');
    removeClass($moonIcon, 'hidden');
};

/* load theme from storage when page loads */
if (localStorage.getItem('theme') === 'dark') {
    applyDarkTheme();
} else if (localStorage.getItem('theme') === 'light') {
    applyLightTheme();
} else {
    /* default theme is dark theme */
    applyDarkTheme();
}

$sunIcon.addEventListener('click', applyLightTheme, false);
$moonIcon.addEventListener('click', applyDarkTheme, false);

function addClass(element, className) {
    element.className += ' ' + className;
}

function removeClass(element, className) {
    const classNameRegEx = new RegExp('\\s*' + className + '\\s*');
    element.className = element.className.replace(classNameRegEx, ' ');
}
```

## Dark theme colors

I like dark themes with blue, purple and yellow contrasts. So I have choosen [this colors](https://coolors.co/0b132b-fee440-f15bb5-000000-ffecd1-390099) for my theme. Many thanks to [Coolors](https://coolors.co/) which has been helping me picking up colors for years 😅.

![Colors](/img/articles/turn-off-the-lights/colors.png)

Of course we have to use this colors in a dedicated `.css` file, to style elements correctly depending on the theme. Below, a short snippet of my `dark.css`:

```css
/************* VARIABLES *************/
body {
    --primary: #0b132b;
    --secondary: #fee440;
    --tertiary: #f15bb5;
    --dark: #000;
    --light: #ffecd1;
    --border: #390099;
}

/************* HEADER *************/
.dark header {
    background-color: var(--dark);
    border-bottom: 1px solid var(--border);
}

.dark .h-wrap h1.title a {
    color: var(--secondary);
}

/************* OFF-CANVAS *************/
.dark .off-canvas.toggled {
    border-left: 1px solid var(--border);
}

.dark .off-canvas {
    background-color: var(--dark);
    color: var(--light);
}

.dark .bio h1 {
    color: var(--secondary);
}

.dark .bio p {
    color: var(--light);
}

/** ... And so forth ... */
```

## Conclusion

Today we have added a dark theme to our hugo blog! Below, a little demonstration

![Example](/img/articles/turn-off-the-lights/example.gif)

If you are interested in `Minhikari`, my fork of `hikari`, you can find it [here](https://github.com/epsxy/minhikari).