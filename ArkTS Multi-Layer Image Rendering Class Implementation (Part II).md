# ArkTS Multi-Layer Image Rendering Class Implementation (Part II)

## I. Advanced Rendering Features

### 1. Rotation & Mirroring
```typescript
public async turn90Area(fromX: number, fromY: number, toX: number, toY: number, counterclockwise?: boolean) {
  const image = this.hiddenContext.getPixelMap(fromX, fromY, toX - fromX, toY - fromY);
  if (image) await image.rotate(90); // GPU-accelerated rotation via system APIs
  // Clear original area and render rotated content
}

public async flipArea(fromX: number, fromY: number, toX: number, toY: number, horizontally?: boolean) {
  const image = await XImageBitmap.cropImageBitmap(...);
  this.hiddenContext.save();
  // Clear original area
  this.hiddenContext.scale(horizontal ? -1 : 1, horizontal ? 1 : -1); // Mirror transformation
  this.hiddenContext.drawImage(image, horizontal ? -toX : fromX, horizontal ? fromY : -toY);
  this.hiddenContext.restore();
}
```

### 2. Cropping
```typescript
public async cropArea(fromX: number, fromY: number, toX: number, toY: number) {
  const img = await XImageBitmap.cropImageBitmap(...);
  // Resize canvas and redraw content
  this.widthPX = toX - fromX;
  this.heightPX = toY - fromY;
  this.hiddenCanvas = new OffscreenCanvas(this.widthPX, this.heightPX);
  this.hiddenContext.drawImage(img, 0, 0);
  img.close(); // Resource cleanup
}
```

### 3. Copy/Paste Operations
```typescript
// Copy selected area to clipboard
this.copyArea(fromX, fromY, toX, toY).then(() => {
  this.setEmptyArea(fromX, fromY, toX, toY); // Cut operation
});

// Paste clipboard content
public pasteArea(fromX: number, fromY: number, content: ImageBitmap) {
  this.hiddenContext.drawImage(content, fromX, fromY);
}
```

### 4. Region Filling
```typescript
public fillArea(fromX: number, fromY: number, toX: number, toY: number, color: string) {
  this.hiddenContext.fillStyle = color;
  this.hiddenContext.fillRect(fromX, fromY, toX - fromX, toY - fromY);
}
```

## II. Image Management

### 1. Image Operations
```typescript
// Basic image placement
public drawImage(img: ImageBitmap, fromX: number, fromY: number) {
  this.hiddenContext.drawImage(img, fromX, fromY);
  img.close(); // Immediate resource release
}

// Advanced image composition
public setImage(img: ImageBitmap, fromX: number, fromY: number) {
  this.hiddenContext.globalCompositeOperation = 'source-in'; // Clipping mask effect
  this.hiddenContext.drawImage(img, fromX, fromY);
  this.hiddenContext.globalCompositeOperation = 'source-over'; // Restore normal mode
}
```

### 2. Image Export
```typescript
// Get raw pixel data
public async getPixelMap() {
  return this.hiddenContext.getPixelMap(0, 0, this.widthPX, this.heightPX);
}

// Export as ImageBitmap
public getImageBitmapWithPromise() {
  const image = this.hiddenCanvas.transferToImageBitmap();
  return XImageBitmap.cropImageBitmap(image, 0, 0, this.widthPX, this.heightPX);
}
```

This implementation completes the professional rendering system with:
- Advanced geometric transformations (rotation/mirroring)
- Precision content manipulation (cropping/copy-paste)
- Efficient image import/export capabilities
- Comprehensive resource management

The architecture demonstrates best practices for HarmonyOS development:
- Leverages system APIs for hardware acceleration
- Implements proper resource cleanup patterns
- Maintains clear separation between core rendering and application logic
- Supports both basic and advanced creative workflows

This system forms the foundation for applications ranging from graphic design tools to image annotation utilities, providing the necessary performance and flexibility for professional-grade implementations.