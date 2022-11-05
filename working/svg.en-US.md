# SVG implementation

| --- | --- |
| Principal | @XGHeaven |
| Start time | 2022-9-28 |
| Status | In Progress |

---

With the development of the times, the use of SVG will be more and more, and it is no longer limited to the rendering of some vector graphics. Some icons have also begun to use SVG to render.

WebF wants to better integrate with the community and reduce the overall access cost. SVG is a key implementation node.

WebF also does not need to implement a complete SVG rendering implementation. It only needs to implement the core capabilities of front-end dependencies while ensuring performance. Do not add entities unless necessary.

## Spec files

- https://www.w3.org/TR/SVG2/
- https://svgwg.org/svg2-draft/

## Roadmap

Since WebF currently has a lot of missing capabilities for SVG, follow the steps below to make up for it:

- dependent
    - [ ] element namespace concept introduced
    - [x] New bridge https://github.com/openwebf/webf/pull/18
    - [x] CSS animation capability https://github.com/openwebf/webf/pull/11
- Basic, meet the most common Icon rendering
    - [ ] SVG parsing
    - [ ] SVG rendering implementation, including how to convert Element to RenderObject, style attribute control, etc.
    - [ ] Provides basic SVG elements, mainly including graphic classes and group classes
    - [ ] Provided by basic SVG Interface, supports direct control via JS
    - [ ] Test the rendering of mainstream front-end frameworks, including Vue/React/Svelte/Angular
- Interaction, functionality
    - [ ] SVG interaction event implementation
    - [ ] SVG supports CSS control, CSS selectors
    - [ ] img element/CSS Background supports asynchronous loading of web SVG
    - [ ] SVG animation, supports CSS animation and `<animation>` element
- Advanced
    - [ ] More and more complex SVG element support
    - [ ] More Attribute support
    - [ ] CSS url/filter function supports #id selector
- Development, optimization
    - [ ] DevTools ability optimization
    - [ ] Performance evaluation, this part may not be completed at one time, and will be added later
    - [ ] CanIUse documentation completion

## design

### namespcaeURI

In the standard, the behavior of an element is actually defined by namespace. The same element with the same tagName but different namespace is still a different element.

So you need to make up for this part of the ability:

- Support xmlns attribute
- Added Element.namespaceURI
- Add document.createElementNS method
- Add Attr.namespaceURI
- Added Element.setAttributeNS method

The implementation needs to modify the logic of the entire createElement, because the current element creation has only one concept of `tagName`, so you need to add the parameter of `namespaceURI`.

The overall logic is not complicated, but there will be many changes.

### SVG parsing

Looking at it now, the current parser of HTML supports parsing SVG, but the corresponding namespace needs to be modified.

After parsing is complete, use the completed createElementNS method to create SVG elements.

### SVG Interface

According to the standard, all SVG elements are inherited from SVGElement, so related interfaces need to be implemented on both the Dart side and the JS side.

List of interfaces: https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model#svg_interfaces

Specification file: https://svgwg.org/svg2-draft/types.html#InterfaceSVGElement

#### extension

At present, Dart's custom components in WebF should not be able to implement dynamic binding types on the JS side, and can be optimized in the follow-up. To reduce the difficulty and pressure of implementing this part of the interface, it only needs to be implemented on the Dart side, and the corresponding types can be automatically created and bound on the JS side.

### SVG rendering

The overall rendering logic is similar to normal Element.

`SVGElement` -> `SVGRenderObject` -> `paint`

- `SVGElement` is the DOM element
- `SVGRenderObject` is the RenderObject corresponding to SVGElement, which is mainly used to handle styles, events, etc. and Element-related things
- `paint` draws all elements

`SVGRenderObject` has two modes:

- Attach mode, which means that it is attached to an Element and can process and respond to styles, events, etc.
- Detach mode, runs independently, does not bind Element instances, cannot respond to styles, can only use default styles, and cannot respond to events. Mainly used to render SVG as background.

### SVG elements

Basic Graphic Elements:

- `<circle>`
- `<line>`
- `<path>`
- `<polygon>`
- `<polyline>`
- `<rect>`
- `<ellipse>`
- `<text>`
- `<image>` TBD

Animated elements:

- `<animatin>`
- `<animateMotion>`
- `<animateTransform>`
- `<mpath>`
- `<set>`

Container element:

- `<svg>`
- `<use>`
- `<a>`
- `<defs>`
- `<g>`
- `<marker>`
- `<mask>`
- ~`<missing-glyph>`~
- `<pattern>`
- `<switch>` TBD
- `<symbol>`
- `<clipPath>`
- `<textPath>`

Gradient elements:

- `<linearGradient>`
- `<radialGradient>`
- `<stop>`

Descriptive elements:

> Empty implementation is the main method, if necessary in the future, it can be further implemented

- `<desc>`
- `<metadata>`
- `<title>`

font element (obsolete): not implemented

Light source element: not implemented

Filter element: TBD recommends that it be determined based on the usage scenario in the future. Currently, it is not required to be implemented.

Other elements:

- `<script>` / `<style>` is not implemented yet, but it needs to be compatible with this situation, so as to prevent the tags inside the SVG from affecting the elements outside


### SVG Events

In theory, SVG events are not much different from DOM events. The main difference lies in the processing of the SVGElement when doing hitTest in the touch event.

And because the drawing of SVG is strictly in accordance with the order of the document, there is no need to consider the issue of sorting, which is generally simpler than the implementation of DOM.

At present, SVG mainly needs to implement the following events:

- [ ] Touch class, such as Touch/Mouse, etc.
- [ ] keyboard events
- [ ] focus event

### SVG and CSS

According to the specification, SVG elements can be styled through CSS, including but not limited to class/style.

Specification file: https://www.w3.org/TR/SVG/styling.html#UsingPresentationAttributes

Specific to the implementation, according to the specification, all `presentation-attributes` can be controlled by the style of the same name, and have the lowest priority. That is to say, as long as there is style/class, `attribute` is invalid.

So in the implementation, you only need to handle `attribute` as the style with the lowest priority, and `getComputedStyle` will also automatically return the result containing `attribute`.

### SVG animation

There are two main sources of SVG animation in HTML, CSS animation and `<animation>` tag.
Most front-end developers mainly use CSS, and a few SVGs use animation tags.

- CSS animation: just follow the current rendering process
- Animation tag: added to the animation of the parent element after converting to CSS Animation.

### Pure image rendering

In addition to being directly used in HTML, SVG can also be rendered as the value of img and background. At this time, SVG only needs to be rendered as an image.

There are a lot of problems that need to be solved at this time.

#### parsing

Since the original parsing method is implemented through the HTML Parser in the bridge, in theory, if you want to directly render SVG to the image, you need to go through:

`Dart String` -> `C String` -> `parse` -> `C GumboNode` -> `JSX Like Array` -> `Dart Node` -> `SVGRenderObject Detach Mode` -> `paint`

The string needs to be copied into C first, and then the parsed structured data is passed back into Dart.
Moreover, it is difficult to send the GumboNode on the C side directly to the Dart side, so it needs to be converted into a data structure of JSX Like Array first. Similar to:

```ts
{
    type: NativeString // Label name
    props: NativeString[] // pair of properties
    childrenCount: number // There are several child elements, traversed in the order of DFS
}[]
````

TBD: However, this solution, the performance is very poor, still needs to be discussed. A better solution is to migrate the HTML parsing logic from C back to Dart.

## Development experience

### DevTools Adaptation

- Support for DevTools inspect
- Support for modifying properties

### Performance Testing

At present, there is no good infrastructure for performance testing, and we can build together by the way.

- SVG performance for HTML strings, including parsing and runtime performance
- SVG performance for JS creation, deletion
- Animation performance
- JS-controlled Animation performance
- Performance when SVG is used as a background image

### CanIUse

Cooperate with the solution proposed by andycall before, try and realize this part of caniuse's ability.

> I have implemented a library here that can directly render compatibility data into a website similar to caniuse.
> In the future, you can try to graft directly onto this library.
> https://github.com/XGHeaven/canyouuse
