Command untuk cari file berisi coding mencurigakan:
```
grep -r -i --include=\*.php 'file_put_contents(' ./ > grep-output-file_put_contents
grep -r -i --include=\*.php 'move_uploaded_file(' ./ > grep-output-move_uploaded_file
grep -r -i --include=\*.php 'base64_decode(' ./ > grep-output-base64_decode
grep -r -i --include=\*.php 'eval(' ./ > grep-output-eval
grep -r -i --include=\*.php "allowed_types'] = '" ./ * > grep-output-allowed_types
grep -r -i --include=\*.php "is_writable(" ./ > grep-output-is_writable

grep -r -i --include=\*.jpeg '<?php' ./ > grep-output-file_jpeg
grep -r -i --include=\*.jpg '<?php' ./ > grep-output-file_jpg
grep -r -i --include=\*.png '<?php' ./ > grep-output-file_png

grep -r -i --include=\*.php 'allowed_types' ./ * > grep-output-allowed_types
```

```
grep -r -i --include=\*.log '112.215.167.61' ./
grep -r -i '112.215.167.61' ./


grep -r -i --include=\*.php 'fileToUpload' ./
grep -r -i --include=\*.php.jpg 'move_uploaded_file(' ./
grep -r -i --include=\*.jpg '<?php' ./
grep -r -i --include=\*.png '<?php' ./
```
