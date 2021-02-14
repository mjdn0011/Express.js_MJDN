### Requirements
1. celine.mp3 라는 파일이 있고, 
   사용자가 /celine.mp3를 방문하는 경우, 서버에서는 인터넷을 통해 이 MP3를 전송해야 함.
2. 사용자가 /burrito.html을 요청하는데 폴더에 그런 파일이 없다면,
   서버에서는 404 에러를 보내야 함.
3. 서버에서 성공 여부에 상관없이 모든 요청을 기록함.
   사용자가 요청한 시간에 요청한 URL을 기록해야 함.

### Middleware Stack
1. **logger: 로깅 미들웨어**
콘솔에 요청된 URL과 요청이 일어난 시간을 출력
2. **static file 송신기: 정적 파일 서버 미들웨어**
폴더에 해당 파일이 존재하는지 여부를 확인.
파일이 존재하면 파일 전송(`res.sendFile`), 
없으면 최종 미들웨어로 실행을 넘김(`next`)
3. **404 handler**
파일이 없는 경우 호출