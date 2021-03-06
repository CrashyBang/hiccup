# Hiccup

A super strict yet probably too lenient methodology for writing maintainable CSS.

<!-- If you want to see it in action demo [here](http://codepen.io/crashy/pen/grBQyp) (intentionally ugly to make it easy to follow). -->

### Intention

To encourage minimal markup and help write concise and maintainable css.

### Why

Why not use just another CSS naming convention like BEM or OOCSS. I found that they were either too dictating, too restrictive, or just made my class names rediculosly huge like `.some-class__child-class--class-modifier` really??? O.o who wants that?

### How

So how does it work? Please bear in mind that this is a living document and that it is subject to change.

There are six kinds of classes used with the hiccup methodology:

* Components
* Sub-components - `*`
* Children - `_` & `~`
* Modifiers - `&`
* States - `+`
* Globals - `!`

**Note**: we explicitly avoid using the `-` to identify classes so it can be used in class names (for example `.side-menu`) without gettign confusing.

We differentiate these class types using prefixes as follows:

**Globals**:

```scss
.\!font-large {
  font-size: 30px;
}
```

Globals are the only selector not typecast and that do not require the direct descendant selector as they can be used globally.

**Components**:

```scss
div.menu {
  background-color: lime;
  display: inline-block;
  min-width: 350px;
  height: 100vh;
  float: left;
}
```

All classes **excluding globals** should be typecast to a certain element like `div.menu` why is this?
When we write CSS we assume certain properties about the element we are going to apply the class to (for example that a `div` is `display: block;`). This means despite how CSS is usually written there is an intended HTML structure for said CSS.
To strictly enforce the intended structure we require all classes be element typecast to force the correct semantics.

As an added bonus this also speeds up the browsers CSS selector engine.

**Children**:

```scss
div.menu {
  //...
  a._menu\~link {
    display: block;
    padding: 10px;
    color: white;
    background-color: deepskyblue;
    border-bottom: 2px dashed white;
    text-decoration: none; 
  }
}
```

Like BEM we identify children using the `_` however we do this slightly differently using the `_` at the start of the class name followed by the parent component name with the `~` appended and then the class name of the child. This is so you can quickly differentiate between a new component and a components child.

**Sub-components**

```scss
div.\*profile {
  //...
  div._\*profile\~image {
    //...
    img {
      //...
    }
  }
}
```

Sub-components are similar to components but with **two** big differences.

**One**: sub-components explicity require that no components or sub-components be nested within them, this means you do not need to accomodate for any unknown markup within your component.

_When is this prudent?_

Usually for elements where you may be utilizing `position: absolute;` (or other more explicit CSS properties) and you know exactly the elements you intended to have in the component and you only want to accomodated for them and no extra.

**Two**: sub-components allow their children to target elements without classes for example:

```scss
div.\*profile {
  //...
  div._\*profile\~image {
    //...
    img {
      //...
    }
    span {
      //...
    }
  }
}
```

_Why bother?_

When I set out to create hiccup one of my biggest frustrations with BEM was for things like a sub-component where I knew nothing would ever be nested within the component but I still had to adhere to giving every element I styled a class name including **wrapping elements** which make for (IMHO) terrible markup. Who wants to read:

```html
<div class="profile">
  <div class="profile__image-wrapper">
    <img class="profile__image" src="..." alt="..."/>
    <span class="profile__link-wrapper">
      <a class="profile__link">
        Title text
      </a>
    </span>
  </div>
</div>
```

With hiccup that becomes the following:

```html
<div class="*profile">
  <div class="_*profile~image">
    <img src="..." alt="..."/>
    <span>
      <a>
        Title Text
      </a>
    </span>
  </div>
</div>
```

Because we know the exact markup our sub-component will contain we are free to target the `img` directly to apply whatever additional styles we need.

If you are adverse to targeting elements without classes that is fine sub-components still utilize children just like regular components. If you were to avoid targeting elements without classes the only difference between your sub-components and your regular components would be that your sub-components would require components and sub-components to not be nested beneath them.

**Modifiers**:

```scss
div.menu {
  //...
  a._menu\~link {
    &.\&active {
      background-color: purple;
    }
  }
}
```

Modifiers will never by typecast because they will always be applied with another class whether it is a component or a child.

**Note**: modifiers can be applied to Components, Sub-components, and Children.

**States**:

```scss
div.menu {
  //...
  &.\+shrunk {
    a._menu\~link {
      //...
    }
  }
}
```

A state works similar to a modifier but it is top level and instead of just modifying the element it is applied to it allows you to modify its children and their modifiers.

**Note**: states can be applied to Components and Sub-components.

<!-- In most cases states will be applied/toggled with javascript (as seen in [demo](http://codepen.io/crashy/pen/grBQyp)) -->

### When should something become a component?

There are two common instances where a child should become a component/sub-component.

1) When you are repeating styles between components:

```scss
div.profile {
  a._profile\~email-link {
    padding: 10px 15px;
    background-color: lime;
    &:hover {
      background-color: deeppink;
    }
  }
}
```

```scss
form.contact-form {
  button._contact-form\~submit {
    padding: 10px 15px;
    background-color: lime;
    &:hover {
      background-color: deeppink;
    }
  }
}
```

Seeing as these styles are the same they can become:

```scss
a, button {
  &.\*button {
    padding: 10px 15px;
    background-color: lime;
    &:hover {
      background-color: deeppink;
    }
  }
}
```

2) When you have up to two children that belong within another child:

```scss
div.certificate {
  div._certificate\~profile {
    //...
  }
  div._certificate\~profile-image {
    //...
  }
  div._certificate\~profile-title {
    //...
  }
}
```

With the above `div._certificate\~profile-image` and `div._certificate\~profile-title` are intended to be nested within `div._certificate\~profile`. This would be okay.

```scss
div.certificate {
  div._certificate\~profile {
    //...
  }
  div._certificate\~profile-image {
    //...
  }
  div._certificate\~profile-title {
    //...
  }
  div._certificate\~profile-content {
    //...
  }
  div._certificate\~profile-social-media {
    //...
  }
}
```

Where as with the above it would probably make sense to extract `profile` out to its own component:

### Why so complicated?

_Wouldn't it be easier to drop the concept of Sub-components and States and move towards a more BEM like structure?_

In short YES. But we won't, why? CSS is global meaning that the level of specificity is up to you. Hiccup is more specific / complicated than most other naming conventions not because they do not have these issues but because they do not address them well. Yes more specific CSS is harder to write but it is also easier to maintain.

### Conclusion

Hiccup is still in it's very early days and there is still plenty of room to improve so any suggetions / edits please fire through a PR.
