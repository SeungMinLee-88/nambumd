chunkUpload: function (url, file, chunkSize = 1024 * 1024, callback) {  
        let uuid = fn_custom.uuid();  
        const totalChunks = Math.ceil(file.size / chunkSize);  
        let chunkIndex = 0;  
        runUpload(url, file, chunkSize, callback, uuid, totalChunks, chunkIndex);  
        function runUpload(url, file, chunkSize, callback, uuid, totalChunks, chunkIndex){  
            const start = chunkIndex * chunkSize;  
            const end = Math.min(start + chunkSize, file.size);  
            const chunk = file.slice(start, end);  
            const formData = new FormData();  
            formData.append('file', chunk, file.name);  
            formData.append("chunk", chunkIndex);  
            formData.append("chunks", totalChunks);  
            formData.append("uuid", uuid); // 업로드 고유 식별자  
  
  
            const key = CryptoJS.enc.Base64.parse("MTExMTIyMjIzMzMzNDQ0NDU1NTU2NjY2Nzc3Nzg4ODg=");  
            const iv  = CryptoJS.enc.Base64.parse("QUJDREVGMTIzNDU2Nzg5MA==");  
  
  
  
            chunk.arrayBuffer().then(function (arrayBuffer) {  
                // 🔥 2) ArrayBuffer -> CryptoJS WordArray                const wordArray = CryptoJS.lib.WordArray.create(arrayBuffer);  
  
                // 🔥 3) AES 암호화  
                const encrypted = CryptoJS.AES.encrypt(wordArray, key, { iv: iv });  
                const ciphertext = encrypted.toString(); // Base64 문자열  
  
                // 🔥 4) JSON으로 전송 (uuid, chunkIndex, totalChunks, fileName + 암호문)  
                const bodyJson = JSON.stringify({  
                    uuid: uuid,  
                    chunk: chunkIndex,  
                    chunks: totalChunks,  
                    fileName: file.name,  
                    data: ciphertext  
                });  
  
                return fetch(url, {  
                    method: "POST",  
                    headers: {  
                        "Content-Type": "multipart/form-data; charset=utf-8"  
                    },  
                    body: bodyJson  
                });  
            }).then(data => {  
                if(data) {  
                    callback(data);  
                }  
            });  
        }  
    },  
    ajaxAbort:function(){  
        fn_customAjax.obj?.abort();  
    }  
}