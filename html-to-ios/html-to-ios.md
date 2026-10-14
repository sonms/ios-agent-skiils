---
name: html-to-ios
description: Convert a Figma design node or an HTML/CSS prototype into idiomatic SwiftUI code. Trigger when a user shares a Figma link or HTML/CSS and asks to implement, build, port, or turn it into an iOS screen or component. Do not use for UIKit unless the user explicitly requests UIKit.
---

# Figma/HTML to SwiftUI

Two inputs, one outcome: an idiomatic SwiftUI implementation that reuses the target project's components and tokens. Treat generated UI as static unless the user also requests state, navigation, or business logic.

## 0. Inspect the target project

Before implementation, find the existing SwiftUI design system: color, typography, spacing, reusable views, asset catalogs, and image-loading conventions. Reuse these first. If none exists, keep the new values semantic and local to the requested scope; do not silently invent a parallel global design system.

## 1. Figma input

- Extract `fileKey` and `nodeId` from the URL. Never guess a missing `node-id`; request a node-specific link.
- Load the available `figma-swiftui` skill, and load `figma-design-to-code` before `get_design_context`. Its React/Tailwind output is a design reference, not SwiftUI source.
- Begin with `get_design_context` for the exact node. Its default screenshot is the visual reference; request `get_screenshot` separately only for another node or a higher-resolution check.
- If the response is large, sparse, or structurally unclear, call `get_metadata`, choose logical child nodes, and fetch those individually. Do not force one huge response.
- Use `get_variable_defs` only when variables or styles are absent or ambiguous in the context response, or when an explicit token mapping is needed.
- When Code Connect data is present, use the mapped component and its documented API instead of inferring a replacement.

## 2. HTML input

Use the HTML/CSS as a visual and structural reference. Split multi-screen files by screen, then apply the token, layout, asset, and verification rules below.

## 3. Tokens and assets

- Map Figma variables and named styles to the target project's color, typography, spacing, and shape tokens. Preserve an unmatched value only when necessary and identify it as a candidate token rather than duplicating raw values throughout the view.
- Figma image and SVG assets take priority. Download supplied assets before their URLs expire, add them through the target's asset-catalog or established image-loading flow, and reference them from SwiftUI. A web-oriented `localhost` URL is not an iOS app asset and must not remain in production Swift code.
- Reuse a provided Figma icon or image rather than adding an icon package or a placeholder. Generate imagery only when no suitable source asset exists and the user has approved any Figma AI credit use.

## 4. Layout translation

| Design pattern | SwiftUI approach |
|---|---|
| Absolute overlay, dim layer, fixed panel | `ZStack` with a semantic sheet, popover, or overlay; avoid reproducing coordinates when layout primitives express the intent. |
| Flex row or column | `HStack` or `VStack` with `alignment` and `spacing`. |
| CSS grid | `LazyVGrid` or `Grid` with semantic column definitions. |
| Centering transforms | `frame(maxWidth: .infinity, alignment: ...)` or container alignment, not offset math. |
| Gap, padding, radius | Existing spacing and shape tokens; otherwise one scoped semantic constant. |
| Fixed control width derived from screen width | Flexible frame plus horizontal padding, not a device-specific width. |
| Repeated siblings differing only in content | A reusable view rendered with `ForEach` over data. |
| Mock OS keyboard, home indicator, or status chrome | Not an app UI target; rely on the platform behavior instead. |

For an icon-only control, provide an accessibility label. Preserve Dynamic Type and avoid using color as the only way to convey state.

## 5. Verify

Compare the implementation with the Figma screenshot for text, hierarchy, color, weight, spacing, asset choice, and behavior implied by the layout. Before completion, confirm there are no expired asset URLs, placeholders where source assets exist, or raw values where a matching project token exists.
