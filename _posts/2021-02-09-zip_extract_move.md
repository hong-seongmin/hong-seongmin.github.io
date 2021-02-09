---
layout: single
title:  "zip압축해제후이동 코드"
---


import os
import zipfile
import shutil
from tkinter import *

# exe 만들기(cmd에서) : (pip install pyinstaller →) cd C:\Users\User\Desktop\파이썬 연습 → pyinstaller -w -F -i=zip압축해제후이동ICON.ico .\zip압축해제후이동.py


def resource_path(relative_path):#절대경로를 상대경로로 변환('pyinstaller -F'에서 필수)
    try:
        base_path = sys._MEIPASS
    except Exception:
        base_path = os.path.abspath(".")
    return os.path.join(base_path, relative_path)

def unzip(source_file, dest_path):
    global SortSelect
    with zipfile.ZipFile(source_file, 'r') as zf:
        NameLists=[]#확인 작업용 zip 내 이름리스트
        ZipInfos = zf.infolist()
        if SortSelect == '시간순':
            ZipInfos = sorted(ZipInfos, key = lambda x:x.date_time)#시간 순 정렬
        elif SortSelect == '이름순':
            ZipInfos = sorted(ZipInfos, key = lambda x:x.filename)#이름 순 정렬
        for ZipInfo in ZipInfos:
            ZipInfo.filename = ZipInfo.filename.encode('cp437').decode('euc-kr', 'ignore')#파일명 한글화
            NameLists.append(ZipInfo.filename.rstrip('/'))#폴더의 경우 뒤에 '/'가 붙으므로 제거해주고 추가
            zf.extract(ZipInfo, dest_path)#압축해제
    return NameLists

root=Tk()
root.title("zip압축해제 설정")

root.geometry("300x500")


NowFolderLabel=Label(root,text="압축을 해제할 폴더 경로 ",bg="yellow").pack()
NowFolderVar=StringVar()#실제로 값을 받는 곳
NowFolder = Entry(root,width=100,textvariable=NowFolderVar)
NowFolder.pack()

MoveFolderLabel=Label(root,text="압축풀린 파일을 옮길 폴더 경로 ",bg="yellow").pack(pady=(30,0))#pady : 위아래간격
MoveFolderVar=StringVar()#실제로 값을 받는 곳
MoveFolder= Entry(root, width=100,textvariable=MoveFolderVar)
MoveFolder.pack()

FileNameLabel=Label(root,text="압축해제 한 파일들 이름 설정",bg="yellow").pack(pady=(30,0))
FileNameSelect=StringVar()
FileName_Keep = Radiobutton(root, text="이름유지",value="이름유지",variable=FileNameSelect)
FileName_Keep.pack()
FileName_Change = Radiobutton(root, text="압축파일기준 이름 변경",value="압축파일기준 이름 변경",variable=FileNameSelect)
FileName_Change.pack()
FileName_Keep.select()#기본값


MoveFileLabel=Label(root,text="압축해제 한 파일들 폴더 설정",bg="yellow").pack(pady=(30,0))#pady : 위아래간격
MoveFileSelect=StringVar()
MoveFile_Divide = Radiobutton(root, text="압축파일기준 폴더 분리", value="압축파일기준 폴더 분리",variable=MoveFileSelect)
MoveFile_Divide.pack()
MoveFile_one = Radiobutton(root, text="한 폴더에 압축해제", value="한 폴더에 압축해제",variable=MoveFileSelect)
MoveFile_one.pack()
MoveFile_Divide.select()#기본값

SortLabel=Label(root,text="이름 변경시 정렬기준",bg="yellow").pack(pady=(30,0))#pady : 위아래간격
SortSelectVar=StringVar()
Sort_Name = Radiobutton(root, text="이름순", value="이름순",variable=SortSelectVar)
Sort_Name.pack()
Sort_Time = Radiobutton(root, text="시간순", value="시간순",variable=SortSelectVar)
Sort_Time.pack()
Sort_Name.select()#기본값



FinishButton=Button(root,text="설정완료",command=lambda:root.destroy())#완료 및 종료 버튼
FinishButton.pack(pady=(30,0))

root.mainloop()
os.chdir(resource_path(NowFolderVar.get()))
TempMoveFolder=resource_path(MoveFolderVar.get())
SortSelect=SortSelectVar.get()

DirList=os.listdir(resource_path(NowFolderVar.get()))#작업중 폴더의 초기 파일들
RealZip=[]#실제 zip파일 목록
# print(DirList)
for IsZip in DirList:#작업중 폴더 내 Zip만 추출
    if os.path.isfile(IsZip):
        if '.zip' in IsZip:
            RealZip.append(IsZip)


for DirZip in RealZip:
    # stories_zip = zipfile.ZipFile(resource_path(NowFolderVar.get()+'\\'+DirZip))#압축 해제할 zip파일 지정
    ZipNameLists=unzip(DirZip,resource_path(NowFolderVar.get()))#압축풀고 한글 인식된 zip 안의 namelist를 return
    NowDirLists=os.listdir(os.getcwd())#압축해제 후 작업중인 폴더리스트
    snum=1#번호매기기 초기값
    for ZipNameList in ZipNameLists:#아까 풀었던 것을 하나하나 살펴봄
        if ZipNameList in NowDirLists:#Zip리스트에 있는 것이 지금 폴더에 있다면
            if not ZipNameList in DirList:#지금 보고 있는 것이 초기 파일이 아니라면
                cfilename = DirZip.split('.zip')[0]#그 zip파일의 이름을 변수에 저장
                MoveFolder=TempMoveFolder#이동할 장소를 아까 저장했던 임시 폴더로 지정
                #파일명 결정
                if os.path.isfile(ZipNameList):#현재 보고 있는 것이 파일이라면
                    cextension = '.'+ZipNameList.split('.')[-1]#변경할 파일의 확장자 확정
                elif os.path.isdir(ZipNameList):#현재 보고 있는 것이 폴더라면
                    cextension=""#확장자 없음
                else:
                    pass
                if FileNameSelect.get()=="이름유지":#압축해제 파일들 이름 결정
                    ChangeName=resource_path(NowFolderVar.get()+'\\'+ZipNameList)#변경할 이름 = 원래 이름
                elif FileNameSelect.get()=="압축파일기준 이름 변경":
                    zeronum="0"*(3-len(str(snum)))#번호 매기기 쉽게 하기 위한 0추가
                    ChangeName=resource_path(NowFolderVar.get()+'\\'+cfilename+" - "+zeronum+str(snum)+cextension)#변경할 파일 이름
                    os.rename(resource_path(NowFolderVar.get()+'\\'+ZipNameList),ChangeName)
                    snum+=1
                #폴더 결정
                if MoveFileSelect.get()=="한 폴더에 압축해제":#압축해제 폴더 결정
                    try:
                        shutil.move(ChangeName, MoveFolderVar.get())#압축해제한 파일 이동
                    except:
                        pass#중복된 파일명일 경우 넘김
                elif MoveFileSelect.get()=="압축파일기준 폴더 분리":
                    cfilename=cfilename.rstrip('.')#폴더 이름의 마지막 부분에 '.'붙어있으면 걍 삭제되므로 오류남
                    NewFolder=resource_path(resource_path(MoveFolderVar.get()) + '\\' + cfilename)#압축파일 기준으로 새로운 폴더 결정
                    os.makedirs(NewFolder, exist_ok=True)#디렉토리 생성(없을 경우에만 생성)
                    try:
                        shutil.move(ChangeName, NewFolder)#압축해제한 파일 이동(긴 path는 오류)
                    except:
                        pass
    # stories_zip.close()#zip닫음
