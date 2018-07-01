# IIIF Presentation API as Entity Component System

This is a proposal (WIP) for an [extension](http://iiif.io/api/annex/registry/extensions/) of the [IIIF Presentation API](http://prezi3.iiif.io/api/presentation/3.0/) to allow it to be used as an [Entity Component System](https://en.wikipedia.org/wiki/Entity%E2%80%93component%E2%80%93system).

`Canvas` is equivalent to `Entity`.

`Annotation` is equivalent to `Component`.

IIIF viewers ('engines' is a more appropriate term?) contain the `System` logic to process the effects of components (annotations) on entities (canvases).

The IIIF-ECS Extension would permit an extra set of annotation `motivations` to be used.

These would allow annotations to specify _behaviours_, confering display properties such as `scale`, `position`, `rotation` with their corresponding json values to canvases.

The proposed IIIF 'engines' would ignore canvas properties of `width`, `height`, and `duration` - using annotated behaviours instead.

Using the `motivation` of `painting` to annotate a 3D resource onto a `canvas` is therefore permitted, shifting authority/responsibility for how media can be displayed from the annotated to the annotator.

To extend a IIIF manifest to allow ECS behaviour, include the custom schema:

```
"@context": [
    "https://edsilv.github.io/iiif-ecs-proposal/ecs.json",
    "http://iiif.io/api/presentation/3/context.json"
]
```

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
        "id": "https://edsilv.github.io/iiif-ecs-proposal/annotations/continuous-images/scale.json",
        "format": "application/json"
    }
}
```

```json
{
    "x": 100,
    "y": 100,
    "z": 0
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
        "id": "https://edsilv.github.io/iiif-ecs-proposal/annotations/3d-transform/position.json",
        "format": "application/json"
    }
}
```

```json
{
    "x": 0,
    "y": 0,
    "z": -1
}
```

Defines the position of the canvas relative to the camera. In this example, centered and 1 unit's distance away.

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
        "id": "https://edsilv.github.io/iiif-ecs-proposal/annotations/3d-transform/rotation.json",
        "format": "application/json"
    }
}
```

```json
{
    "x": 45,
    "y": 90,
    "z": 180
}
```

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
        "id": "https://edsilv.github.io/iiif-ecs-proposal/annotations/continuous-images/display.json",
        "format": "application/json"
    }
}
```

```json
{
    "viewingDirection": "top-to-bottom",
    "continuous": true
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
        "id": "https://edsilv.github.io/iiif-ecs-proposal/annotations/auto-advancing-audio/playback.json",
        "format": "application/json"
    }
}
```

```json
{
    "duration": 3723.4,
    "continuous": true
}
```

The playback component adds `duration` and other temporal properties to a `canvas`.

The `continuous` property in this context instructs the playback `system` to advance to the next playable `entity` when this `entity`'s playable `duration` ends.

</details>


### References

https://www.slideshare.net/workergnome/extending-iiif-30
https://w3c.github.io/web-annotation/model/wd/#extending-motivations

<!--
## Notes

Does the `painting` motivation still make sense? Does one 'paint' a non-visual audio file onto a canvas? Perhaps something like `asset` is more generic?

Three.js would allow [2D](https://threejs.org/docs/#api/cameras/OrthographicCamera) or [3D](https://threejs.org/docs/#api/cameras/PerspectiveCamera) presentation. Perhaps use a `camera` component with a `projection` value of `orthographic` or `perspective`?

aframe (written in three.js) has a complete ECS implementation already. Work would be required to map IIIF ECS components to [aframe components](https://github.com/aframevr/aframe/tree/master/docs/components).
-->
