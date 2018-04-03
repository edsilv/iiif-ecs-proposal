# IIIF Presentation API as Entity Component System

This is a proposal (WIP) for how to use the [IIIF Presentation API](http://prezi3.iiif.io/api/presentation/3.0/) as an [Entity Component System](https://aframe.io/docs/0.8.0/introduction/entity-component-system.html).

`Canvas` is equivalent to `Entity`.

`Annotation` is equivalent to `Component`.

IIIF viewers contain the `System` logic.

To extend a IIIF manifest to allow ECS behaviour, include a custom schema, e.g.

```
"@context": [
    "http://iiif.io/api/presentation/3/context.json",
    "http://customschema.com/ecs.json"
]
```

Use custom annotation `motivation`s to classify components.

## Components

<details>
<summary>Scale</summary>

```json
{
    "id": "https://edsilv.github.io/iiif-ecs-proposal/continuous-images.json/items/canvas/1/annotation/1",
    "type": "Annotation",
    "motivation": "scale",
    "target": "https://edsilv.github.io/iiif-ecs-proposal/continuous-images.json/items/canvas/1",
    "body": {
        "x": 100,
        "y": 100,
        "z": 0
    }
}
```

[StringBody](https://www.w3.org/TR/annotation-model/#string-body) allows values to be included directly into annotations. 
Could the ecs schema override `body` to permit complex json data values within annotations? The content of these complex objects would be defined per component type.

A component with a `motivation` of `scale` would accept a body containing only `x`, `y`, and `z` values.

In the example above, the `x`, `y`, and `z` values describe a flat plane with width and height of 100. This is equivalent to a conventional 2D image.

</details>

<details>
<summary>Position</summary>

```json
{
    "id": "https://edsilv.github.io/iiif-ecs-proposal/3d-transform.json/items/canvas/0/annotation/2",
    "type": "Annotation",
    "motivation": "position",
    "target": "https://edsilv.github.io/iiif-ecs-proposal/3d-transform.json/items/canvas/0",
    "body": {
        "x": 0,
        "y": 0,
        "z": -1
    }
}
```

Defines the position of the canvas relative to the camera. In this example, centered and 1 unit's distance away.

</details>

<details>
<summary>Display</summary>

```json
{
    "id": "https://edsilv.github.io/iiif-ecs-proposal/continuous-images.json/items/canvas/3/annotation/2",
    "type": "Annotation",
    "motivation": "display",
    "target": "https://edsilv.github.io/iiif-ecs-proposal/continuous-images.json/items/canvas/2",
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

</details>

<details>
<summary>Playback</summary>

```json
{
    "id": "https://edsilv.github.io/iiif-ecs-proposal/auto-advancing-audio.json/items/canvas/0/annotation/1",
    "type": "Annotation",
    "motivation": "playback",
    "target": "https://edsilv.github.io/iiif-ecs-proposal/auto-advancing-audio.json/items/canvas/0",
    "body": {
        "duration": 3723.4,
        "continuous": true
    }
}
```

The playback component adds `duration` and other temporal properties to a `canvas`.

The `continuous` property in this context instructs the playback `system` to advance to the next playable `entity` when this `entity`'s playable `duration` ends.

</details>

<details>
<summary>Rotation</summary>

```json
{
    "id": "https://edsilv.github.io/iiif-ecs-proposal/3d-transform.json/items/canvas/0/annotation/3",
    "type": "Annotation",
    "motivation": "rotation",
    "target": "https://edsilv.github.io/iiif-ecs-proposal/3d-transform.json/items/canvas/0",
    "body": {
        "x": 45,
        "y": 90,
        "z": 180
    }
}
```

</details>

## Notes

Does the `painting` motivation still make sense? Does one 'paint' a non-visual audio file onto a canvas? Perhaps something like `asset` is more generic?

Three.js would allow [2D](https://threejs.org/docs/#api/cameras/OrthographicCamera) or [3D](https://threejs.org/docs/#api/cameras/PerspectiveCamera) presentation. Perhaps use a `camera` component with a `projection` value of `orthographic` or `perspective`?

aframe (written in three.js) has a complete ECS implementation already. Work would be required to map IIIF ECS components to [aframe components](https://github.com/aframevr/aframe/tree/master/docs/components).
