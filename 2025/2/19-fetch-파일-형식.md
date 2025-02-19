```javascript
       const content = Buffer.alloc(3 * 1024 * 1024); // 3MB 크기 버퍼 생성
         const mockFile = new Blob([content], {
             type: 'text/plain'
        });
        mockFile.name = 'test.txt';

       const mockFile = new Blob([fileBuffer], {
            type: 'video/mp4'  // MP4의 MIME 타입
        });
        mockFile.name = 'file.mp4';

        //다른 타입
        type: 'video/x-m4v'  // M4V 파일의 MIME 타입

```
