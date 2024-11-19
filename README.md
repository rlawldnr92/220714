function sendEmail() {
    const ss = SpreadsheetApp.getActiveSpreadsheet()
    const sheet1 = ss.getSheetByName('list'); // 이메일 보낼 사람들 목록이 들어간 시트명!
    const sheet2 = ss.getSheetByName('template'); // 이메일 내용이 들어간 시트명!
    const subject = sheet2.getRange(2,1).getValue(); // 이메일 제목 추출
    const n = sheet1.getLastRow(); // 이메일 보낼 명단이 전체 몇 명인지 확인
    const folder = DriveApp.getFolderById('1y6IpVRnuWV2EFWdAbiUBshus8TNlAG1y'); // 폴더 id를 확인해서 추가
    const files = folder.getFiles();
    const fileMap = {};
    while (files.hasNext()) {
      let file = files.next();
      // 맥에서 작업해서 한글이 자소 분리된 경우, 다시 합쳐버린다.
      const fileName = file.getName().replace(/\s/g, '');
      const encodeFileName = Utilities.base64Encode(fileName, Utilities.Charset.UTF_8);
      // Map을 활용해서 폴더 내 파일을 파일이름으로 분류한다.
      fileMap[encodeFileName] = file;
    }
    
    // 2번째 행부터 이메일 주소가 있기때문에 2번째 행부터 시작해서 마지막 줄까지 반복하는 구문
    for (let i = 2; i < n+1 ; i++ ) {
        // 이메일 주소는 2번째 열에 있으니까 (i,2)
        const emailAddress = sheet1.getRange(i,2).getValue(); 
        // 이름은 첫번째 열에 있으니까!! (i, 1)
        const name = sheet1.getRange(i,1).getValue();

        // 이메일 제목에서 <name>이라고 써진 부분을 갈아치운다
        const updateSubject = subject.replace("<name>",name); 
        
        // 이메일 내용이 될부분 추출
        let message = sheet2.getRange(2,2).getValue();
        // <name> 이라고 써진 부분을 실제 이름으로 갈아치우는 코드!
        message = message.replace("<name>",name);

        // 보내야할 파일을 특정해서, attachments array로 만들어서 보내보자.
        const excelFileName = sheet1.getRange(i,3).getValue().replace(/\s/g, '');
        const pdfFileName = sheet1.getRange(i,4).getValue().replace(/\s/g, '');
        const encodeExcelFileName = Utilities.base64Encode(excelFileName, Utilities.Charset.UTF_8);
        const encodePdfFileName = Utilities.base64Encode(pdfFileName, Utilities.Charset.UTF_8);
        const excelFile = fileMap[encodeExcelFileName];
        const pdfFile = fileMap[encodePdfFileName];
        const attachments = [excelFile, pdfFile];
        
        MailApp.sendEmail({
          to: emailAddress,
          subject: updateSubject,
          body: message,
          attachments: attachments,
        });
        
    }
}
