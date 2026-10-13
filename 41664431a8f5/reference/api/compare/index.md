---
title: "Compare API"
description: "ImageTolerance and the Scion.Compare library - Image, NormRect, PixelRect, Mask, ImageDiff, DiffStatus and DiffResult - for perceptual image comparison."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/compare/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api/compare.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/image-comparison.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/raw-frames.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Compare API

`ImageTolerance` (in `Scion.Runtime`) states how much two captures may differ. `Scion.Compare` (in the package and on
NuGet, no Unity needed) loads PNG and raw frames, builds masks, compares images with OKLab and HyAB and writes
heatmaps. JSON and PNG coordinates have the origin at the top left.

## ImageTolerance

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** none

```csharp
public sealed class ImageTolerance
{
    public ImageTolerance(float pixel, float maxRatio, int maxPixels);
    public static readonly ImageTolerance Default;   // (0.1, 0.001, -1)
    public static readonly ImageTolerance Exact;     // (0, 0, 0)
    public float Pixel { get; }
    public float MaxRatio { get; }
    public int MaxPixels { get; }                    // -1 = no limit
}
```

## Image

**Namespace:** `Scion.Compare` · **Assembly:** `Scion.Compare` · **Define:** none

```csharp
public sealed class Image
{
    public int Width { get; }
    public int Height { get; }
    public byte[] Rgba { get; }                      // RGBA8, top row first
    public static Image Load(string path);           // .png, or .raw with raw.json beside it
    public static Image FromRaw(byte[] raw, int width, int height, string layout, bool bottomUp);
    public void SavePng(string path);
}
```

## NormRect

**Namespace:** `Scion.Compare` · **Assembly:** `Scion.Compare` · **Define:** none

```csharp
public readonly struct NormRect
{
    public NormRect(float x, float y, float w, float h);   // 0 to 1, origin top left
}
```

## PixelRect

**Namespace:** `Scion.Compare` · **Assembly:** `Scion.Compare` · **Define:** none

```csharp
public readonly struct PixelRect
{
    public int X { get; }
    public int Y { get; }
    public int Width { get; }
    public int Height { get; }
}
```

## Mask

**Namespace:** `Scion.Compare` · **Assembly:** `Scion.Compare` · **Define:** none

```csharp
public sealed class Mask
{
    public static Mask Empty(int width, int height);
    public static Mask FromRects(int width, int height, IEnumerable<NormRect> rects);
    public static bool TryLearn(IReadOnlyList<Image> samples, ImageTolerance t, float cap, out Mask mask, out float unstable);
    public float Coverage { get; }
    public Mask Union(Mask other);
    public static Mask Load(string png);
    public void Save(string png);
}
```

`TryLearn` returns false (and no mask) when the unstable area exceeds `cap`; it never truncates.

## DiffStatus

**Namespace:** `Scion.Compare` · **Assembly:** `Scion.Compare` · **Define:** none

```csharp
public enum DiffStatus { Same, Within, Changed }
```

`new` and `stale` are CLI verdicts, not image statuses.

## DiffResult

**Namespace:** `Scion.Compare` · **Assembly:** `Scion.Compare` · **Define:** none

```csharp
public readonly struct DiffResult
{
    public DiffStatus Status { get; }
    public double Ratio { get; }
    public int Pixels { get; }
    public PixelRect Bounds { get; }
}
```

## ImageDiff

**Namespace:** `Scion.Compare` · **Assembly:** `Scion.Compare` · **Define:** none

```csharp
public static class ImageDiff
{
    public static DiffResult Compare(Image a, Image b, Mask mask, ImageTolerance t, bool parallel = false);
    public static void WriteHeatmap(Image a, Image b, Mask mask, ImageTolerance t, string pngPath);
}
```

## See also

- [Image comparison](/scion/41664431a8f5/trials/image-comparison/)
- [Raw frames](/scion/41664431a8f5/reference/files/raw-frames/)
- [Baselines and approval](/scion/41664431a8f5/trials/baselines/)
