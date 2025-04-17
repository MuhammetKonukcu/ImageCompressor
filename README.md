# ImageCompressor

A lightweight Android utility for efficiently compressing images while preserving quality. Implemented as a Kotlin singleton object, `ImageCompressor` handles resizing, sampling, EXIF-based rotation, and dynamic quality selection based on file size.

## 🚀 Usage

Call `compress` from a coroutine scope (e.g., `lifecycleScope`) since it is a `suspend` function performing IO operations:

```kotlin
import com.example.utils.ImageCompressor

lifecycleScope.launch {
    val uriString: String = /* the URI string of the image to compress */
    val compressedPath: String? = ImageCompressor.compress(
        context = requireContext(),
        uriString = uriString
    )

    if (compressedPath != null) {
        // The absolute path of the compressed JPEG file
        println("Compressed image saved at: $compressedPath")
    } else {
        // Compression failed
        Toast.makeText(requireContext(), "Image compression failed", Toast.LENGTH_SHORT).show()
    }
}
```

**Note:** `compress` runs on the IO dispatcher. Always call it within a coroutine.

## ✨ Features

* **Dimension Scaling:** Resizes images to fit within `1920×1080` while preserving aspect ratio.
* **Sampling Optimization:** Calculates an optimal `inSampleSize` to reduce memory usage.
* **High-Quality Scaling:** Uses `Canvas` and `Matrix` for smooth bitmap scaling.
* **EXIF Rotation:** Reads EXIF orientation data to rotate images correctly.
* **Dynamic Quality Selection:** Adjusts JPEG compression quality based on the original file size.
* **Automatic Naming:** Generates unique filenames using timestamps and random suffixes.
* **Simple API:** Single `compress` method for easy integration.

## ⚙️ Dynamic Quality via `chooseQuality`

The `chooseQuality` function selects a JPEG quality level based on the input file size:

| File Size | JPEG Quality |
|-----------|-------------|
| > 5 MB | 40 |
| 2–5 MB | 50 |
| 1–2 MB | 60 |
| 0.5–1 MB | 70 |
| 0.25–0.5 MB | 80 |
| ≤ 0.25 MB | 90 |

You can modify `chooseQuality` to suit your project's requirements by changing size thresholds or quality values:

```kotlin
private fun chooseQuality(context: Context, uri: Uri): Int {
    val size = queryFileSize(context, uri)
    return when {
        size > 10_000_000 -> 30  // Lower quality for files > 10 MB
        size > 5_000_000  -> 40
        size > 2_000_000  -> 55  // Adjust threshold or quality here
        else              -> 85  // Higher base quality for small files
    }
}
```

## Contributions and pull requests are welcome! 🌟
