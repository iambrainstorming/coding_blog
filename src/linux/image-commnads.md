# Image commands

```bash
sudo snap install pdftk

pdftk file1.pdf file2.pdf cat output mergedfile.pdf


pdftk full-pdf.pdf cat 12-15 output outfile_p12-15.pdf
```

## svg to pdf

```bash
sudo apt-get install librsvg2-bin

rsvg-convert -f pdf -o foo.pdf foo.svg


inkscape "$pdf" -z --export-dpi=600 --export-area-drawing --export-png="$pngfile"
```

## images to pdf

```bash
convert image.jpg image2.jpg image3.jpg  output.pdf
```


# Pdf to images

```bash
sudo apt-get install poppler-utils

pdftocairo -png file.pdf  image_file.png
```

# Crop video
It crops from 01:00:00.0 than 2 minutes.
```bash
ffmpeg -ss 01:00:00.0 -i video.mp4 -t 00:02:00 -c copy cropped_video.mp4
```
