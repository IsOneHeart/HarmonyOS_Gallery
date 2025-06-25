# HarmonyOS Special Compressed Package Format Import&Export Implementation Guide

This document uses the Pixel Artisan APP's pixel art project file format (.pap) as an example to deeply analyze technical solutions for implementing special compressed package file management in HarmonyOS, covering core functional details such as file compression, decompression, and sharing.

## I. Technical Background and File Format Design

The .pap file format, serving as Pixel Artisan's project format, is based on ZIP compression algorithm and stores project files through a specific directory structure. Its technical implementation includes three core modules:

1. **File Export Module**: Compresses project files into standard ZIP format
2. **File Import Module**: Decompresses ZIP and reconstructs project directory
3. **System Sharing Module**: Transfers files through HarmonyOS sharing mechanism

## II. File Compression Export Implementation

```typescript
export function exportPAP(inFile: string, outFile: string) {
  let options: zlib.Options = {};
  zlib.compressFile(inFile, outFile, options).then((data: void) => {
    console.info('Compression successful, data: ' + JSON.stringify(data));
    return true;
  }).catch((errData: BusinessError) => {
    console.error(`Compression error code: ${errData.code}, message: ${errData.message}`);
    return false;
  })
}
```

## III. File Decompression Import Implementation

```typescript
export async function importPAP(inFile: string, outFile: string): Promise<boolean> {
  let flag: boolean = true;
  try {
    fs.mkdirSync(outFile);
  } catch (e) {
    // Directory already exists or other errors
  }
  await zlib.decompressFile(inFile, outFile);
  return flag;
}
```

## IV. System Sharing Integration

```typescript
export function sharePAP(filePath: string, title: string, thumbnail: Uint8Array, context: common.UIAbilityContext) {
  // Construct ShareData with valid data configuration
  // Get accurate UTD type
  // let utdTypeId = uniformTypeDescriptor.getUniformDataTypeByMIMEType('zip-archive/pap');
  let utdTypeId = utd.getUniformDataTypeByFilenameExtension('.zip', utd.UniformDataType.ZIP_ARCHIVE);
  let shareData: systemShare.SharedData = new systemShare.SharedData({
    utd: utdTypeId,
    uri: fileUri.getUriFromPath(filePath),
    title: title, // File name displayed when title not provided
    // description: 'Image description', // File size displayed when description not provided
    thumbnail: thumbnail // Priority uses provided thumbnail preview, defaults to original image if not provided
  });
  // Display sharing panel
  let controller: systemShare.ShareController = new systemShare.ShareController(shareData);
  controller.show(context, {
    selectionMode: systemShare.SelectionMode.SINGLE,
    previewMode: systemShare.SharePreviewMode.DETAIL,
  }).then(() => {
    console.info('Sharing panel displayed successfully.');
  }).catch((error: BusinessError) => {
    console.error(`Sharing panel display error. Code: ${error.code}, Message: ${error.message}`);
  });
}
```

This implementation demonstrates how to create specialized file formats while maintaining compatibility with standard system components in HarmonyOS development.