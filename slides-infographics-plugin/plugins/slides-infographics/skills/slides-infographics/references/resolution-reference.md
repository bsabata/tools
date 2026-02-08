# Resolution and Aspect Ratio Reference

## Resolution by Aspect Ratio

| Aspect Ratio | 1K Resolution | 2K Resolution | 4K Resolution | Best For              |
|--------------|---------------|---------------|---------------|-----------------------|
| 16:9         | 1376x768      | 2752x1536     | 5504x3072     | Slides (default)      |
| 9:16         | 768x1376      | 1536x2752     | 3072x5504     | Vertical infographics |
| 1:1          | 1024x1024     | 2048x2048     | 4096x4096     | Social media cards    |
| 3:2          | 1264x848      | 2528x1696     | 5056x3392     | Wide infographics     |
| 2:3          | 848x1264      | 1696x2528     | 3392x5056     | Tall infographics     |
| 21:9         | —             | —             | —             | Ultra-wide timelines  |
| 4:3          | —             | —             | —             | Classic slide format   |
| 3:4          | —             | —             | —             | Portrait print format  |

**Note:** Aspect ratios 21:9, 4:3, and 3:4 are accepted by the API but exact pixel dimensions per resolution tier are not documented. The API determines the appropriate resolution automatically for these ratios.

## Recommendations

- **Presentation slides**: Use 16:9 at 2K (2752x1536). Sharp on all modern displays and projectors.
- **Vertical infographics**: Use 9:16 at 2K for scrollable web or social story content.
- **Social media cards**: Use 1:1 at 1K for square-format posts.
- **Ultra-wide timelines**: Use 21:9 at 2K for horizontal timeline infographics.
- **Print output**: Use 3:4 or 2:3 at 4K for high-DPI print materials.
- **General rule**: Use 2K unless you specifically need print quality (4K) or are constrained by file size (1K).
