# IIIF Presentation API as Entity Component System

This is a proposal (WIP) for how to use the [IIIF Presentation API](http://prezi3.iiif.io/api/presentation/3.0/) as an [Entity Component System](https://aframe.io/docs/0.8.0/introduction/entity-component-system.html).

`Canvas` is equivalent to `Entity`.

`Annotation` is equivalent to `Component`.

IIIF viewers contain the `system` logic.

To extend your IIIF manifest to allow ECS behaviour, include a custom schema:

```
"@context": [
    "http://iiif.io/api/presentation/3/context.json",
    "http://customschema.com/ecs.json"
]
```

Use custom annotation `motivation`s to classify components.

## Components

<details>
<summary>Scale Component</summary>

```js
{
    "id": "https://edsilv.github.io/strawmen/behavior/mixed.json/items/canvas/2/annotation/1",
    "type": "Annotation",
    "motivation": "scale",
    "target": "https://edsilv.github.io/strawmen/behavior/mixed.json/items/canvas/2",
    "body": {
        "x": 100,
        "y": 100,
        "z": 0
    }
}
```

`width` and `height` as direct properties of a canvas implies that a resource can always have spatial dimensions. Audio files for example do not. 3D files would also require an extra `depth` value. I propose a `scale` component to encode these values if appropriate for the resource.

[StringBody](https://www.w3.org/TR/annotation-model/#string-body) sets a precendent for including values directly into annotations. I propose that the ecs schema overrides `body` to permit complex json data values typed per component.

A component with a `motivation` of `scale` would accept a body containing only `x`, `y`, and `z` values.

In the example above, the `x`, `y`, and `z` properties describe a flat plane with width and height of 100. This is equivalent to a conventional 2D image.

</details>

## Position Component

```
{
    "id": "https://edsilv.github.io/strawmen/behavior/transform.json/items/canvas/0/annotation/2",
    "type": "Annotation",
    "motivation": "position",
    "target": "https://edsilv.github.io/strawmen/behavior/transform.json/items/canvas/0",
    "body": {
        "x": 0,
        "y": 1,
        "z": -1
    }
}
```



## Display Component

```
{
    "id": "https://edsilv.github.io/strawmen/behavior/mixed.json/items/canvas/3/annotation/2",
    "type": "Annotation",
    "motivation": "display",
    "target": "https://edsilv.github.io/strawmen/behavior/mixed.json/items/canvas/2",
    "body": {
        "viewingDirection": "top-to-bottom",
        "continuous": true
    }
}
```

The `continuous` `viewingHint` requires the presence of a `viewingDirection` in IIIF. I propose that these are consolidated into properties of a single `display` component per `canvas`.

In a 3D context, a `viewingDirection` of `top-to-bottom` could imply stacking on the z index. Maybe add `near-to-far`, `far-to-near` to remove ambiguity?

`viewingDirection` could have a 'sensible default' of `left-to-right`, `continuous` of `false`.

If `continuous` is `false`, is that equivalent to stacking on the z axis? i.e. `viewingDirection:near-to-far`?

## Playback Component

```
{
    "id": "https://edsilv.github.io/strawmen/behavior/mixed.json/items/canvas/0/annotation/1",
    "type": "Annotation",
    "motivation": "playback",
    "target": "https://edsilv.github.io/strawmen/behavior/mixed.json/items/canvas/0",
    "body": {
        "duration": 3723.4,
        "continuous": true
    }
}
```

## Rotation Component

```
{
    "id": "https://edsilv.github.io/strawmen/behavior/transform.json/items/canvas/0/annotation/3",
    "type": "Annotation",
    "motivation": "rotation",
    "target": "https://edsilv.github.io/strawmen/behavior/transform.json/items/canvas/0",
    "body": {
        "x": 45,
        "y": 90,
        "z": 180
    }
}
```

The playback component adds `duration` and other strictly temporal properties to a `canvas`.

The `continuous` property in this context instructs the playback `system` to advance to the next playable `entity` when this `entity`'s playable `duration` ends.

## Notes

Does the `painting` motivation still make sense? Does one 'paint' a non-visual audio file onto a canvas? Perhaps something like `asset` is more generic?

Three.js would allow [2D](https://threejs.org/docs/#api/cameras/OrthographicCamera) or [3D](https://threejs.org/docs/#api/cameras/PerspectiveCamera) presentation. 

aframe (written in three.js) has a complete ECS implementation already. Work would be required to map IIIF ECS components to aframe components: https://github.com/aframevr/aframe/tree/master/docs/components
