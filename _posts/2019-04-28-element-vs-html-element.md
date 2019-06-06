---
title: Element vs HTMLElement
---

Let's consider a `Node` as the basic and generic object that represents everything which appears in a DOM tree.
Node is a html tag, comment, an html element. Or let me put that the other way around - every DOM object inherits from Node. 
<br>
<br>
``` html
<div id="note">
  <!--Who am I?-->
  <p>Name: Max</p>
  <p>Description of myself!</p>
</div>
```
``` javascript
const element = document.getElementById('note');
element.toString(); //=> HTMLDivElement

const elementList = document.querySelectorAll('p');
elementList.toString(); //=> NodeList
elementList[0].toString(); //=> HTMLParagraphElement

const tags = document.getElementsByTagName('p');
tags.toString(); //=> HTMLCollection, list of HTMLElements
tags[0].toString(); //=> HTMLParagraphElement

element instanceof Node //=> true 
element instanceof Element //=> true
```

<br>
An `Element` is a possible incarnation of a Node. So every Element is a Node, but not every Node is an Element.
`<!--Who am I?-->` comment is a Node, but neither an Element nor an HTMLElement. Every node has own type and id ([see all node types with its ids](https://developer.mozilla.org/en-US/docs/Web/API/Node/nodeType#Node_type_constants)).
<br>
<br>

``` javascript
element.nodeType //=> 1 - it's a Node.ELEMENT_NODE with id 1
element.childNodes //=> NodeList(7) [text, comment, text, p, text, p, text]

// lets grab <!--Who am I?--> comment
element.childNodes[1].nodeType //=> 8 - it's a Node.COMMENT_NODE with id 8
element.childNodes[1].nodeType.data //=> "Who am I?"

element.childNodes[1] instanceof Element //=> false
element.childNodes[1] instanceof Comment //=> true
```
<br>
Ok, but let's go back to the topic that includes `HTMLElement`. What's the difference between Element and HTMLElement.
Both are Nodes. Both are Elements. The difference is that not every Element is an HTMLElement. _Element can refer to objects different from HTML universe i.e to XML DOM element._
<br>
<br>

``` xml
<!-- note.xml file -->
<?xml version="1.0" encoding="UTF-8" ?>
<note>
 <name>Name: Max</name>
 <description>Description of myself!</description>
</note>
```

``` javascript
// fetching xml 
const xhttp = new XMLHttpRequest();

xhttp.onreadystatechange = function() {
  if (this.readyState == 4 && this.status == 200) {
    // check instances of xml elements
    showInsanceOfElement(this)
  }
};

xhttp.open("GET", "note.xml", true);
xhttp.send();

function showInsanceOfElement(xml) {
  xmlDoc = xml.responseXML;
  const [xmlElement] = xmlDoc.getElementsByTagName('note');
    
  xmlElement instanceof Element //=> true
  xmlElement instanceof HTMLElement //=> false
}
```
<br>
### Why do the difference is important? React `ref`'s on Typescript
<br>

Imagine a React component written in Typescript that renders element with `React.createElement` function from top-level React API with a `ref` callback.
<br>
<br>

``` typescript
class CustomComponent extends React.Component<{}> {
  private customElement?: HTMLElement | null
  // customElement is the union type (HTMLElement or null)
    
  render() {
    return React.createElement('div', {
      ref: element => this.customElement = element
      // element instanceof Element //=> true
      // element instanceof HTMLElement //=> true or false 
    }
  }
}
```
<br>
Unfortunately, in this case, Typescript raises an error - `Element is not assignable to HTLMElement`. 
When I started working with DOM elements and Typescript I couldn't understand this kind of behaviour since
I had considered every Element as an HTMLElement before, but it's not true indeed. An element could be an 
HTMLElement but not only. Element has a wider interface scope and in Typescript, we have to take care of inheritance structure.
That's why a DOM element should be cast or type-guarded (here to HTMLElement or null as in React ref can be also null if the component is unmounted).
The difference becomes important if you use these Typescript concepts. 
<br>
<br>

``` typescript
ref: element => this.customElement = element as HTMLElement | null; 
```
<br>
Adding a cast is always a code smell but it's not critical here because ref element (if exists) is an HTMLElement.
Currently, Typescript is completely sure that every `element` inside this if-block has HTMLElement 
interface (or is null) and it won't raise an error any longer.