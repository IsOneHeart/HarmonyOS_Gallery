# ArkTS Multi-Layer Image Rendering Class Implementation (Part I)

This article analyzes the implementation of a professional 2D rendering system for HarmonyOS using object-oriented design principles, based on actual production code. Part I focuses on core system architecture and fundamental rendering capabilities.

## I. Dual-Buffer Rendering Engine Architecture

### 1. Off-Screen Canvas System
```typescript
  constructor(layerNo: number, layerName: string, widthPX: number, heightPX: number,
    content?: ImageBitmap, viewable?: boolean, lock?: boolean) {
    this.layerNo = layerNo;
    this.layerName = layerName;
    this.canView = true;
    this.lock = false;
    this.widthPX = widthPX;
    this.heightPX = heightPX;
    this.hiddenCanvas = new OffscreenCanvas(widthPX, heightPX, LengthMetricsUnit.PX);
    this.hiddenContext = this.hiddenCanvas.getContext("2d", this.settings);
    //this.visualCanvas=new CanvasRenderingContext2D(settings)
    if (content) {
      this.hiddenContext.drawImage(content, 0, 0);
    }
    if (viewable !== null && viewable !== undefined) {
      this.canView = viewable;
    }
    if (lock !== null && lock !== undefined) {
      this.lock = this.lock;
    }
  }
```
- **Dual Buffering Mechanism**: Utilizes `OffscreenCanvas` for non-main-thread rendering
- **Anti-aliasing Control**: `RenderingContextSettings(false)` disables anti-aliasing for pixel-perfect precision
- **Context Isolation**: Each layer maintains independent rendering context for complex composition operations

### 2. Layer Management System
```typescript
@ObservedV2
export class Layers {
  @Trace data: Layer[] = [];
}
```
- **Reactive Layer Stack**: Uses `@ObservedV2` for automatic UI updates on layer changes
- **Layer Properties**: Encapsulates canvas dimensions, visibility, and lock state
- **Rendering Order**: Controlled by array sequence (later layers render on top)

## II. Core Rendering Capabilities

### 1. Basic Rendering Operations
#### Brush System
```typescript
  public async draw(pxX: number, pxY: number, color: string, weight: number) {
    //await this.clear(pxX, pxY, weight, true);
    this.hiddenContext.fillStyle = color;
    if ((pxX === this.lastPointX && pxY === this.lastPointY) || this.lastPointX === -1 || this.lastPointY === -1) {
      this.hiddenContext.globalCompositeOperation = 'destination-out';
      this.hiddenContext.fillStyle = '#ffffffff';
      switch (weight) {
        case 1:
          this.hiddenContext.fillRect(pxX, pxY, 1, 1)
          break
        case 2:
          this.hiddenContext.fillRect(pxX, pxY, 2, 2)
          break
        case 3:
          this.hiddenContext.fillRect(pxX - 1, pxY - 1, 3, 3)
          break
        case 4:
          this.hiddenContext.fillRect(pxX - 1, pxY, 4, 2)
          this.hiddenContext.fillRect(pxX, pxY - 1, 2, 4)
          break
        case 5:
          this.hiddenContext.fillRect(pxX - 2, pxY - 1, 5, 3)
          this.hiddenContext.fillRect(pxX - 1, pxY - 2, 3, 5)
          break
      }
      this.hiddenContext.globalCompositeOperation = 'source-over';
      this.hiddenContext.fillStyle = color;
      switch (weight) {
        case 1:
          this.hiddenContext.fillRect(pxX, pxY, 1, 1)
          break
        case 2:
          this.hiddenContext.fillRect(pxX, pxY, 2, 2)
          break
        case 3:
          this.hiddenContext.fillRect(pxX - 1, pxY - 1, 3, 3)
          break
        case 4:
          this.hiddenContext.fillRect(pxX - 1, pxY, 4, 2)
          this.hiddenContext.fillRect(pxX, pxY - 1, 2, 4)
          break
        case 5:
          this.hiddenContext.fillRect(pxX - 2, pxY - 1, 5, 3)
          this.hiddenContext.fillRect(pxX - 1, pxY - 2, 3, 5)
          break
      }
      // this.hiddenContext.beginPath();
      // this.hiddenContext.arc(pxX, pxY, weight/2, 0, 2 * Math.PI);
      // this.hiddenContext.stroke();
      // this.hiddenContext.fill();
      // this.hiddenContext.closePath();
    } else {
      this.hiddenContext.globalCompositeOperation = 'destination-out';
      this.shape(0, this.lastPointX, this.lastPointY, pxX, pxY, '#ffffffff', weight, false, false);
      this.hiddenContext.globalCompositeOperation = 'source-over';
      this.shape(0, this.lastPointX, this.lastPointY, pxX, pxY, color, weight, false, false);
    }
    this.setPoint(pxX, pxY);
  }
```
- **Composite Modes**: Implements transparency effects through `globalCompositeOperation`
- **Dynamic Brush System**: Adjusts rendering area automatically based on weight (1-5 levels)
- **Path Caching**: Maintains last coordinates for continuous line drawing

#### Vector Shape Rendering
```typescript
  public shape(type: number, fromX: number, fromY: number, toX: number, toY: number, color: string, weight: number,
    fillShape?: boolean, first: boolean = true) {
    if(first){
      this.hiddenContext.globalCompositeOperation = 'destination-out';
      this.shape(type, fromX, fromY, toX, toY, color, weight, fillShape, false);
      this.hiddenContext.globalCompositeOperation = 'source-over';
    }
    this.hiddenContext.beginPath()
    switch (type) {
      case 0:
        this.hiddenContext.moveTo(fromX, fromY)
        this.hiddenContext.lineTo(toX, toY)
        break
      case 1:
        this.hiddenContext.rect(fromX, fromY, toX - fromX, toY - fromY)
        break
      case 2: {
        let radius = Math.sqrt(Math.pow(toX - fromX, 2) + Math.pow(toY - fromY, 2)) / 2
        let centerX = (fromX + toX) / 2
        let centerY = (fromY + toY) / 2
        this.hiddenContext.arc(centerX, centerY, radius, 0, 2 * Math.PI, false)
        break
      }
      case 3: {
        let x = (fromX + toX) / 2
        let y = (fromY + toY) / 2
        let w = Math.abs(toX - fromX) / 2
        let h = Math.abs(toY - fromY) / 2
        this.hiddenContext.ellipse(x, y, w, h, 0, 0, 2.1 * Math.PI)
        break
      }
    }
    this.hiddenContext.lineWidth = weight
    this.hiddenContext.strokeStyle = color
    this.hiddenContext.stroke()
    if (!(type === 0) && fillShape) {
      this.hiddenContext.fillStyle = color
      this.hiddenContext.fill()
    }
    this.hiddenContext.closePath()
  }
```
- **Shape Variety**: Supports lines, rectangles, circles, and ellipses
- **Fill Control**: Boolean parameter controls shape filling
- **Style Management**: Centralized control through strokeStyle configuration

### 2. Advanced Rendering Features
#### Flood Fill Algorithm
```typescript
  public async splash(pxX: number, pxY: number, color: string, maxDepth = 32, first: boolean = true) {
    //promptAction.showToast({message:CJNative().hello_cangjie('FUCK')})
    if (first) {
      this.hiddenContext.globalCompositeOperation = 'destination-out';
      this.splash(pxX, pxY, '#ffffffff', 32, false);
      this.hiddenContext.globalCompositeOperation = 'source-over';
    }
    const targetColor = this.hiddenContext.getImageData(pxX, pxY, 1, 1).data;
    const rgb = __XColorData__.autoStr2rgba(color);
    let pixelsToFill: [number, number, number][] = [[pxX, pxY, 0]];
    const checkedPixels = new Set<string>();
    while (pixelsToFill.length > 0) {
      let currentPixel = pixelsToFill.shift();
      if (!currentPixel) {
        continue;
      }
      let currentX = currentPixel[0];
      let currentY = currentPixel[1];
      let currentDepth = currentPixel[2];
      const key = `${currentX},${currentY}`;
      if (checkedPixels.has(key) || currentDepth >= maxDepth) {
        continue;
      }
      checkedPixels.add(key);
      const currentColor = this.hiddenContext.getImageData(currentX, currentY, 1, 1).data;
      if (currentColor[0] === targetColor[0] &&
        currentColor[1] === targetColor[1] &&
        currentColor[2] === targetColor[2] &&
        currentColor[3] === targetColor[3]) {
        if (currentX > 0) {
          pixelsToFill.push([currentX - 1, currentY, currentDepth + 1]);
        } // Left
        if (currentX < this.widthPX - 1) {
          pixelsToFill.push([currentX + 1, currentY, currentDepth + 1]);
        } // Right
        if (currentY > 0) {
          pixelsToFill.push([currentX, currentY - 1, currentDepth + 1]);
        } // Top
        if (currentY < this.heightPX - 1) {
          pixelsToFill.push([currentX, currentY + 1, currentDepth + 1]);
        } // Bottom
        this.hiddenContext.fillStyle = color;
        this.hiddenContext.fillRect(currentX, currentY, 1, 1);
      }
    }
  }
```
- **Color Matching**: Retrieves target pixel color via `getImageData`
- **Depth Limiting**: Prevents infinite recursion with maxDepth parameter
- **Performance Optimization**: Uses Breadth-First Search (BFS) for efficient filling

#### Eraser Functionality
```typescript
  public async clear(pxX: number, pxY: number, weight: number, autoBeforeDrawing: boolean = false) {
    this.hiddenContext.globalCompositeOperation = 'destination-out';
    if ((pxX === this.lastPointX && pxY === this.lastPointY) || this.lastPointX === -1 || this.lastPointY === -1) {
      switch (weight) {
        case 1:
          this.hiddenContext.fillRect(pxX, pxY, 1, 1)
          break
        case 2:
          this.hiddenContext.fillRect(pxX, pxY, 2, 2)
          break
        case 3:
          this.hiddenContext.fillRect(pxX - 1, pxY - 1, 3, 3)
          break
        case 4:
          this.hiddenContext.fillRect(pxX - 1, pxY, 4, 2)
          this.hiddenContext.fillRect(pxX, pxY - 1, 2, 4)
          break
        case 5:
          this.hiddenContext.fillRect(pxX - 2, pxY - 1, 5, 3)
          this.hiddenContext.fillRect(pxX - 1, pxY - 2, 3, 5)
          break
      }
    } else {
      this.shape(0, this.lastPointX, this.lastPointY, pxX, pxY, '#ffffffff', weight, false, false);
    }
    this.hiddenContext.globalCompositeOperation = 'source-over';
    if (!autoBeforeDrawing) {
      this.setPoint(pxX, pxY);
    }
  }
```
- **Precision Erasing**: Selective region clearing through composite modes
- **Weight Adaptation**: Automatically adjusts erasing area based on brush size

## III. Layer Management Basics

```typescript
// Visibility Control
public setCanView(canView?: boolean) {
  this.canView = (canView !== undefined) ? canView : !this.canView;
}

// Layer Locking
public setLock(lock?: boolean) {
  this.lock = (lock !== undefined) ? lock : !this.lock;
}
```
- **State Toggle**: Supports both direct setting and toggle modes
- **Property Encapsulation**: Maintains controlled state access through getters/setters