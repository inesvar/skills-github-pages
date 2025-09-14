---
title: "Mermaid flowgraphs"
date: 2025-09-11
category: embedded-graphics
tag: rust
---

# Legend

```mermaid
graph LR
    S1[[struct]]
    S2[[struct]]
    S3[[struct]]
    S4[[struct]]

    T1([trait:trait_function])
    T2([trait])
    T3([trait])

    I1@{ shape: text, label: "abstract description" }
    I2@{ shape: text, label: "abstract description (this looks better when not drawing the actual objects)" }
    I3@{ shape: text, label: "abstract description" }

    subgraph is_implemented_by
        T1 === S1 & I1
    end

    subgraph is_composed_of
        S3 --o S4 & I2
    end

    subgraph uses
        T2 -.-> S2 & T3 & I3
    end
```

# StyledDrawable `draw_styled`

```mermaid
graph LR
    %% structs
    PS[[PrimitiveStyle]]
    Pri@{ shape: text, label: "Primitives: line, circle, rectangle, etc." }
    
    %% traits
    DT([DrawTarget:draw_iter])
    SD([StyledDrawable:draw_styled])
    PC([PixelColor])

    %% modules
    pixelcolor[pixelcolor]
    primitives[primitives]
    draw_target[draw_target]

    %%subgraph embedded-graphics
        subgraph draw_target
            DT
        end

        subgraph primitives
            PS
            SD
            Pri
        end

        subgraph pixelcolor
            PC
        end
    %%end

    DT -.-> PC
    SD -.-> PS
    SD -.-> DT
    PS -.-> PC
    SD === Pri
```

# Drawable `draw`

```mermaid
graph LR
    %% structs
    S[[Styled]]
    Px[[Pixel]]
    Sty@{ shape: text, label: "Styles: red fill, green 5px outline, etc." }
    Pri@{ shape: text, label: "Primitives: line, circle, rectangle, etc." }
    I[[Image]]
    T[[Text]]
    
    %% traits
    DT([DrawTarget:draw_iter])
    PC([PixelColor])
    D([Drawable:draw])
    P([Primitive:into_styled])

    %% modules
    pixelcolor[pixelcolor]
    primitives[primitives]
    draw_target[draw_target]
    image[image]
    text[text]

    %%subgraph embedded-graphics
        subgraph draw_target
            DT
        end

        subgraph image
            I
        end

        subgraph text
            T
        end

        subgraph primitives
            S ~~~ P
            S --o Sty
            P === Pri
			P -.-> Sty
            S --o Pri
        end

        subgraph pixelcolor
            PC
        end

        D
        Px
    %%end

    DT -.-> PC
    D === S
    D === Px
    D === I
    D === T
    D -.-> DT
```