---
layout: page
title: Square Limit by M.C. Escher
---

![Escher](https://raw.githubusercontent.com/Abhiroop/Abhiroop.github.io/01b52606b3a75bd26574bc22cb36eb6d39383c35/art/sqlimit.svg)

This is a digital rendition of [M.C. Escher's square limit](https://www.wikiart.org/en/m-c-escher/square-limit) artwork. The most surprising thing about this piece
is that it is composed of only four basic tiles shown below:

![Escher Tiles](https://raw.githubusercontent.com/Abhiroop/Abhiroop.github.io/eed5bc0c912e0cff256c677415675b1be26228f7/art/tile.svg)

The entirety of this art is generated by taking these four tiles and using the power of Functional Programming à la function composition to create several powerful combinators that can render the infinite shapes. This technique was illustrated by Peter Henderson in his 1982 seminal paper - [Functional Geometry](https://dl.acm.org/doi/10.1145/800068.802148). The paper mentions Mary Sheeran (who happened to be my PhD supervisor!) as the original implementor in UCSD Pascal. I have implemented the same in Haskell using the wonderful [diagrams](https://hackage.haskell.org/package/diagrams) library.

The code intentionally avoids complicated rasterization and related techniques from computer graphics, so that the simple (yet powerful) essence of *functional geometry* is evident at a glance. The heart of the logic is this:

```haskell
squareLimit :: Diagram B
squareLimit = cycle corner

corner = nonet corner2 side2 side2
               (rot side2) uTile (rot tTile)
               (rot side2) (rot tTile) (rot qTile)

nonet p1 p2 p3 p4 p5 p6 p7 p8 p9
  = onTopOf 1 2  (nextTo 1 2 p1 (nextTo 1 1 p2 p3))
   (onTopOf 1 1  (nextTo 1 2 p4 (nextTo 1 1 p5 p6))
                 (nextTo 1 2 p7 (nextTo 1 1 p8 p9)))

nextTo m n d1 d2 = scaledD1 ||| scaledD2
  where
    scaledD1 = d1 # scaleX (m / total)
    scaledD2 = d2 # scaleX (n / total)
    total = m + n

onTopOf m n d1 d2 = scaledD1 === scaledD2
  where
    scaledD1 = d1 # scaleY (m / total)
    scaledD2 = d2 # scaleY (n / total)
    total = m + n

side2   = quartet side1 side1 (rot tTile) tTile
corner2 = quartet corner1 side1 (rot side1) uTile

uTile = cycle $ rot qTile
tTile = quartet pTile qTile rTile sTile

cycle p1 = quartet p1 (rot $ rot $ rot p1) (rot p1) (rot $ rot p1)

quartet p q r s = scale (1/2) $ centerXY ((p ||| q) === (r ||| s))

rot p = rotate (90 @@ deg) p

pTile = makeTile markingsP
qTile = makeTile markingsQ
rTile = makeTile markingsR
sTile = makeTile markingsS
```

The complete executable code is available [here](https://github.com/Abhiroop/geofunc). To the best of my knowledge, this is one of the best illustrations of the power of function composition to create complex software. There is not a single wasted line, and the essence of the code is as clear and declarative as possible.