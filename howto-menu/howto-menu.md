project_path: /web/_project.yaml
book_path: /web/fundamentals/_book.yaml

{# wf_updated_on: 2017-08-10#}
{# wf_published_on: 2017-04-06 #}

# HowTo: Components – howto-menu {: .page-title }

{% include "web/_shared/contributors/ewagasperowicz.html" %}
{% include "web/_shared/contributors/robdodson.html" %}
{% include "web/_shared/contributors/surma.html" %}

<link rel="stylesheet" href="main.css">

## Summary {: #summary }

A menu is a widget that offers a list of choices to the user,
such as a set of actions or functions. A menu is usually opened,
or made visible, by activating a menu button, choosing an item in a menu
that opens a sub menu, or by invoking a command, such as Shift + F10 in
Windows, that opens a context specific menu.

The element that opens the menu is referenced with aria-labelledby.

See: https://www.w3.org/TR/wai-aria-practices-1.1/#menu


## Demo {: #demo }
{% framebox height="auto" width="100%" class="demo" suppress_site_styles="true" %}
<!doctype html>

<style>
  howto-menu {
    position: absolute;
    display: block;
    width: 10em;
  }

  [role="menuitem"] {
    display: block;
    width: 100%;
    padding: .2em .5em;
    border: 1px solid lightgrey;
  }

  howto-menu:not([opened]) {
    display: none;
  }
</style>

<div class="text">
   <button id="menu-btn">options</button>
   <howto-menu aria-labelledby="menu-btn">
     <span role="menuitem">first option</span>
     <span role="menuitem">second option</span>
     <span role="menuitem">other option</span>
     <span role="menuitem">yet another option</span>
   </howto-menu>
</div>

<script>
  var menu = document.querySelector('howto-menu');
  document.getElementById('menu-btn').addEventListener('click', e => {
    menu.opened = true;
  })
</script>


<script src="https://cdn.rawgit.com/webcomponents/custom-elements/master/custom-elements.min.js"></script>
<script src="https://cdn.rawgit.com/webcomponents/shadydom/master/shadydom.min.js"></script>
<script>
  devsite.framebox.AutoSizeClient.initAutoSize(true);
  (function() {
    /**
 * Copyright 2017 Google Inc. All rights reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
(function() {
  /**
   * Define key codes to help with handling keyboard events.
   */
  const KEYCODE = {
    A: 65,
    DOWN: 40,
    ESCAPE: 27,
    TAB: 9,
    UP: 38,
    Z: 90,
  };

  /**
   * Aria roles corresponding to a menu item.
   */
  const MENUITEM_ARIA_ROLES = [
    'menuitem',
    'menuitemcheckbox',
    'menuitemradio',
  ];

  class HowtoMenu extends HTMLElement {
    static get observedAttributes() {
      return ['opened'];
    }

    /**
     * Returns the first child which is a menuitem.
     */
    _firstMenuItem() {
      let child = this.firstElementChild;
      while (child) {
        if (this._isMenuItem(child)) {
          return child;
        }
        child = next.nextElementSibling;
      }
      return null;
    }

    /**
     * Returns the last child which is a menuitem.
     */
    _lastMenuItem() {
      let child = this.lastElementChild;
      while (child) {
        if (this._isMenuItem(child)) {
          return child;
        }
        child = next.previousElementSibling;
      }
      return null;
    }

    /**
     * Checks if a node is a 'menuitem', 'menuitemcheckbox' or 'menuitemradio'.
     */
    _isMenuItem(node) {
      return MENUITEM_ARIA_ROLES.includes(node.getAttribute('role'));
    }

    /**
     * Returns a menu item coming after the one passed as an argument, or null.
     */
    _nextMenuItem(node) {
      let next = node.nextElementSibling;
      while (next) {
        if (this._isMenuItem(node)) {
          return next;
        }
        next = next.nextElementSibling;
      }
      return null;
    }

    /**
     * Returns a menu item coming before the one passed as an argument, or null.
     */
    _previousMenuItem(node) {
      let prev = node.previousElementSibling;
      while (prev) {
        if (this._isMenuItem(node)) {
          return prev;
        }
        prev = prev.previousElementSibling;
      }
      return null;
    }

    /**
     * Handles keyboard interaction.
     */
    _onKeyDown(event) {
      let el = event.target;
      switch (event.keyCode) {
        case KEYCODE.DOWN:
          // If arrow down, move to next item. Wrap if necessary.
          event.preventDefault();
          let next = this._nextMenuItem(el) || this._firstMenuItem();
          next.focus();
          break;
       case KEYCODE.UP:
         // If arrow up, move to previous item. Wrap if necessary.
         event.preventDefault();
         let prev = this._previousMenuItem(el) || this._lastMenuItem();
         prev.focus();
         break;
       case KEYCODE.ESCAPE:
         // If escape, close the menu.
         event.preventDefault();
         this.opened = false;
         break;
       case KEYCODE.TAB:
         // If tab, close the menu.
         event.preventDefault();
         this.opened = false;
         break;
       default:
         break;
      }
      // If letter key, move to an item which starts with that letter.
      if (event.keyCode >= KEYCODE.A && event.keyCode <= KEYCODE.Z) {
        for (let child of this.children) {
          if (child.innerText.trim()[0] === event.key) {
            event.preventDefault();
            child.focus();
            break;
          }
        }
      }
    }

    _open() {
      this._addTabindex();
      let first = this._firstMenuItem();
      first && first.focus();
    }

    set opened(isOpened) {
      if (Boolean(isOpened)) {
        this.setAttribute('opened', 'true');
      } else {
        this.removeAttribute('opened');
      }
    }

    get opened() {
      return this.hasAttribute('opened');
    }

    _upgradeProperty(prop) {
      if (this.hasOwnProperty(prop) || this.hasAttribute(prop)) {
        let value = this[prop];
        delete this[prop];
        this[prop] = value;
      }
    }

    _onFocusOut(event) {
      if (event.relatedTarget === null || !this.contains(event.relatedTarget)) {
        this.opened = false;
      }
    }

    /**
     * `connectedCallback` fires when the element is inserted into the DOM.
     * It's a good place to set the initial `role`, `tabindex`, internal state,
     * and install event listeners.
     */
    connectedCallback() {
      if (!this.hasAttribute('role')) {
        this.setAttribute('role', 'menu');
      }

      // A user may set a property on an _instance_ of an element,
      // before its prototype has been connected to this class.
      // The `_upgradeProperty` method will check for any instance properties
      // and run them through the proper class setters.
      // See the [lazy properites](#lazy-properties) section for more details.
      this._upgradeProperty('opened');

      this.addEventListener('keydown', this._onKeyDown);
      this.addEventListener('focusout', this._onFocusOut);
    }

    /**
     * Unregisters the event listeners that were set up in `connectedCallback`.
     */
    disconnectedCallback() {
      this.removeEventListener('keydown', this._onKeyDown);
      this.removeEventListener('focusout', this._onKeyDown);
    }

    /**
     * `attributeChangedCallback` is called when any of the attributes in the
     * `observedAttributes` array are changed. It's a good place to handle
     * side effects, like changing the state of the widget.
     */
    attributeChangedCallback(name, oldValue, newValue) {
      if (name === 'opened') {
        if (Boolean(newValue)) {
          this._open();
        } else {
          this._removeTabindex();
        }
      }
    }

    /**
     * Adds tabindex to all menu items to make them selectable.
     */
    _addTabindex() {
      for (let child of this.children) {
        child.setAttribute('tabindex', '0');
      }
    }

    /**
     * Removes tabindex from all menu items to make them unselectable.
     */
    _removeTabindex() {
      for (let child of this.children) {
        child.removeAttribute('tabindex');
      }
    }
  }

  window.customElements.define('howto-menu', HowtoMenu);
})();

  })();
</script>
</html>

{% endframebox %}

## Example usage {: #usage }
<ul class="literate demo" id="howto-menu_demo">

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code ">&lt;!doctype html&gt;

&lt;style&gt;
<span class="indent">&nbsp;&nbsp;</span>howto-menu {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>position: absolute;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>display: block;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>width: 10em;
<span class="indent">&nbsp;&nbsp;</span>}

<span class="indent">&nbsp;&nbsp;</span>[role="menuitem"] {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>display: block;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>width: 100%;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>padding: .2em .5em;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>border: 1px solid lightgrey;
<span class="indent">&nbsp;&nbsp;</span>}

<span class="indent">&nbsp;&nbsp;</span>howto-menu:not([opened]) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>display: none;
<span class="indent">&nbsp;&nbsp;</span>}
&lt;/style&gt;

&lt;div class="text"&gt;
<span class="indent">&nbsp;&nbsp;</span> &lt;button id="menu-btn"&gt;options&lt;/button&gt;
<span class="indent">&nbsp;&nbsp;</span> &lt;howto-menu aria-labelledby="menu-btn"&gt;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> &lt;span role="menuitem"&gt;first option&lt;/span&gt;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> &lt;span role="menuitem"&gt;second option&lt;/span&gt;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> &lt;span role="menuitem"&gt;other option&lt;/span&gt;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> &lt;span role="menuitem"&gt;yet another option&lt;/span&gt;
<span class="indent">&nbsp;&nbsp;</span> &lt;/howto-menu&gt;
&lt;/div&gt;

&lt;script&gt;
<span class="indent">&nbsp;&nbsp;</span>var menu = document.querySelector('howto-menu');
<span class="indent">&nbsp;&nbsp;</span>document.getElementById('menu-btn').addEventListener('click', e =&gt; {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>menu.opened = true;
<span class="indent">&nbsp;&nbsp;</span>})
&lt;/script&gt;</code></pre>
</li>

</ul>

## Code {: #code }
<ul class="literate code" id="howto-menu_impl">
  
<li class="blockcomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code ">(function() {</code></pre>
</li>

<li class="linecomment empty">
<div class="literate-text empty"></div>
<pre><code class="literate-code empty"></code></pre>
</li>

<li class="blockcomment ">
<div class="literate-text "><p>Define key codes to help with handling keyboard events.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span>const KEYCODE = {</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>A: 65,
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>DOWN: 40,
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>ESCAPE: 27,
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>TAB: 9,
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>UP: 38,
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>Z: 90,
<span class="indent">&nbsp;&nbsp;</span>};</code></pre>
</li>

<li class="blockcomment ">
<div class="literate-text "><p>Aria roles corresponding to a menu item.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span>const MENUITEM_ARIA_ROLES = [</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>'menuitem',
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>'menuitemcheckbox',
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>'menuitemradio',
<span class="indent">&nbsp;&nbsp;</span>];

<span class="indent">&nbsp;&nbsp;</span>class HowtoMenu extends HTMLElement {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>static get observedAttributes() {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>return ['opened'];
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}</code></pre>
</li>

<li class="blockcomment ">
<div class="literate-text "><p>Returns the first child which is a menuitem.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>_firstMenuItem() {</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>let child = this.firstElementChild;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>while (child) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>if (this._isMenuItem(child)) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>return child;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>child = next.nextElementSibling;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>return null;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}</code></pre>
</li>

<li class="blockcomment ">
<div class="literate-text "><p>Returns the last child which is a menuitem.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>_lastMenuItem() {</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>let child = this.lastElementChild;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>while (child) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>if (this._isMenuItem(child)) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>return child;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>child = next.previousElementSibling;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>return null;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}</code></pre>
</li>

<li class="blockcomment ">
<div class="literate-text "><p>Checks if a node is a &#39;menuitem&#39;, &#39;menuitemcheckbox&#39; or &#39;menuitemradio&#39;.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>_isMenuItem(node) {</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>return MENUITEM_ARIA_ROLES.includes(node.getAttribute('role'));
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}</code></pre>
</li>

<li class="blockcomment ">
<div class="literate-text "><p>Returns a menu item coming after the one passed as an argument, or null.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>_nextMenuItem(node) {</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>let next = node.nextElementSibling;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>while (next) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>if (this._isMenuItem(node)) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>return next;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>next = next.nextElementSibling;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>return null;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}</code></pre>
</li>

<li class="blockcomment ">
<div class="literate-text "><p>Returns a menu item coming before the one passed as an argument, or null.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>_previousMenuItem(node) {</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>let prev = node.previousElementSibling;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>while (prev) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>if (this._isMenuItem(node)) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>return prev;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>prev = prev.previousElementSibling;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>return null;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}</code></pre>
</li>

<li class="blockcomment ">
<div class="literate-text "><p>Handles keyboard interaction.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>_onKeyDown(event) {</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>let el = event.target;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>switch (event.keyCode) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>case KEYCODE.DOWN:</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text "><p> If arrow down, move to next item. Wrap if necessary.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>event.preventDefault();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>let next = this._nextMenuItem(el) || this._firstMenuItem();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>next.focus();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>break;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> case KEYCODE.UP:</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text "><p> If arrow up, move to previous item. Wrap if necessary.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> event.preventDefault();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> let prev = this._previousMenuItem(el) || this._lastMenuItem();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> prev.focus();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> break;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> case KEYCODE.ESCAPE:</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text "><p> If escape, close the menu.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> event.preventDefault();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> this.opened = false;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> break;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> case KEYCODE.TAB:</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text "><p> If tab, close the menu.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> event.preventDefault();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> this.opened = false;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> break;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> default:
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span> break;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text "><p> If letter key, move to an item which starts with that letter.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>if (event.keyCode &gt;= KEYCODE.A && event.keyCode &lt;= KEYCODE.Z) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>for (let child of this.children) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>if (child.innerText.trim()[0] === event.key) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>event.preventDefault();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>child.focus();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>break;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}

<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>_open() {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>this._addTabindex();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>let first = this._firstMenuItem();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>first && first.focus();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}

<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>set opened(isOpened) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>if (Boolean(isOpened)) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>this.setAttribute('opened', 'true');
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>} else {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>this.removeAttribute('opened');
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}

<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>get opened() {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>return this.hasAttribute('opened');
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}

<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>_upgradeProperty(prop) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>if (this.hasOwnProperty(prop) || this.hasAttribute(prop)) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>let value = this[prop];
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>delete this[prop];
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>this[prop] = value;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}

<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>_onFocusOut(event) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>if (event.relatedTarget === null || !this.contains(event.relatedTarget)) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>this.opened = false;
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}</code></pre>
</li>

<li class="blockcomment ">
<div class="literate-text "><p><code>connectedCallback</code> fires when the element is inserted into the DOM.
It&#39;s a good place to set the initial <code>role</code>, <code>tabindex</code>, internal state,
and install event listeners.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>connectedCallback() {</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>if (!this.hasAttribute('role')) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>this.setAttribute('role', 'menu');
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text "><p> A user may set a property on an <em>instance</em> of an element,
 before its prototype has been connected to this class.
 The <code>_upgradeProperty</code> method will check for any instance properties
 and run them through the proper class setters.
 See the <a href="#lazy-properties">lazy properites</a> section for more details.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>this._upgradeProperty('opened');

<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>this.addEventListener('keydown', this._onKeyDown);
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>this.addEventListener('focusout', this._onFocusOut);
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}</code></pre>
</li>

<li class="blockcomment ">
<div class="literate-text "><p>Unregisters the event listeners that were set up in <code>connectedCallback</code>.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>disconnectedCallback() {</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>this.removeEventListener('keydown', this._onKeyDown);
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>this.removeEventListener('focusout', this._onKeyDown);
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}</code></pre>
</li>

<li class="blockcomment ">
<div class="literate-text "><p><code>attributeChangedCallback</code> is called when any of the attributes in the
<code>observedAttributes</code> array are changed. It&#39;s a good place to handle
side effects, like changing the state of the widget.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>attributeChangedCallback(name, oldValue, newValue) {</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>if (name === 'opened') {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>if (Boolean(newValue)) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>this._open();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>} else {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>this._removeTabindex();
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}</code></pre>
</li>

<li class="blockcomment ">
<div class="literate-text "><p>Adds tabindex to all menu items to make them selectable.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>_addTabindex() {</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>for (let child of this.children) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>child.setAttribute('tabindex', '0');
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}</code></pre>
</li>

<li class="blockcomment ">
<div class="literate-text "><p>Removes tabindex from all menu items to make them unselectable.</p>
</div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>_removeTabindex() {</code></pre>
</li>

<li class="linecomment ">
<div class="literate-text empty"></div>
<pre><code class="literate-code "><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>for (let child of this.children) {
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>child.removeAttribute('tabindex');
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span><span class="indent">&nbsp;&nbsp;</span>}
<span class="indent">&nbsp;&nbsp;</span>}

<span class="indent">&nbsp;&nbsp;</span>window.customElements.define('howto-menu', HowtoMenu);
})();</code></pre>
</li>

</ul>
