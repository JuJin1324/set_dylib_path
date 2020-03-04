# set_dylib_path
macOS용 C 프로그램을 배포 시에 프로그램이 참조할 동적 라이브러리인 .dylib 파일의 위치를 지정하는 방법 정리

## dylib
macOS에서 동적 라이브러리 파일은 dylib 확장자를 갖는다.

### 의존성 확인
curl 라이브러리가 의존하는 라이브러리들 목록 확인 예시) 
```bash
cd /usr/local/Cellar/curl-openssl/7.68.0/lib
otool -L libcurl.dylib
```

### 빌드 후 만들어질 프로그램이 참조할 라이브러리 위치 확인
예시) `otool -D libcurl.dylib`

결과 예시) 
`libcurl.dylib:
/usr/local/opt/curl-openssl/lib/libcurl.4.dylib`

위와 같이 libcurl.dylib를 링크하는 프로그램은 다음의 경로에 libcurl.4.dylib 가 있어야 참조하여 실행이 가능하다.

### 빌드 후 만들어질 프로그램이 참조할 라이브러리 위치 변경
동적 라이브러리 링크를 통해서 빌드한 프로그램을 배포 시에 우리가 각 컴퓨터 마다 brew install 을 통해서 libcurl를 해당 경로에 
설치할 순 없는 노릇임으로 임의로 배포할 프로그램의 external/lib 아래에 libcurl.4.dylib 를 가져다놓는다고 가정하자.
```
배포 root
├── 배포 프로그램
└── external
    └── lib
        └── libcurl.4.dylib
```

1. `/usr/local/opt/curl-openssl/lib/libcurl.4.dylib` 에 존재하는(설치 경로는 사용자마다 다름으로 필요한 라이브러리 경로는 알아서 찾자.) 
libcurl.4.dylib 를 해당 라이브러리를 사용하는 프로젝트의 임의의 디렉터리 아래에 넣어둔 후에 터미널로 해당 디렉터리로 이동한다.

경로 예시)
```
프로젝트 root
├── 여러 개발 파일들...
└── external
    └── Darwin
        └── lib
            └── libcurl.4.dylib
```

2. libcurl.4.dylib 가 있는 디렉터리로 이동 후(프로젝트 root/external/Darwin/lib) 
`install_name_tool -id "./external/lib/libcurl.4.dylib" libcurl.4.dylib` 를 통하여 빌드로 산출된 프로그램의 위치에서 
external 디렉터리를 찾아 그 아래 lib 디렉터리 아래 libcurl.4.dylib 라이브러리를 실행 시 참조하도록 설정을 완료하였다.

3. 빌드 프로그램이 실행시 라이브러리 참조 위치 확인
`otool -L [빌드프로그램 이름]` 을 통하여 확인 가능
