# University_homework
chmod +x ./http_server* 명령어를 사용해 http_server로 시작하는 이름의 파일들에 실행 권한 부여
./http_server_<YOUR_SYSTEM> 62123 명령어를 이용해 서버를 켠다 (YOUR_SYSTEM : apple_silicom, intel_mac, linux, WSL)
http://127.0.0.1:62123 으로 웹페이지 접근!

## Application 1: HTTP server

    typedef struct http_field_t
    {
    
        char *field;
        char *val;
    } http_field_t;
---
    typedef struct http_t
    {
    
        char *method;
        char *path;
        char *version;
        char *status;
    
        size_t body_size;
        void *body_data;
    
        int field_count;
        int max_field_count;
        http_field_t *fields;
    } http_t;
---
    http_t *init_http ()

- http_t *http calloc으로 생성, http NULL이면 에러 출력 NULL 반환.
- http_t 내 모든 요소 0, NULL, default로 초기화. field는 malloc. NULL이면 에러 출력 NULL 반환.
---
    http_t *init_http_with_arg (char *method, char *path, char *version, char *status)

- version과 status NULL이면 에러 출력 NULL 반환.
- http_t *response 생성 및 init_http()로 초기화. NULL이면 에러 출력 NULL 반환.
- 인자로 받은 모든 요소 copy_string()으로 복사하며 response 각 요소에 연결. NULL시 에러 출력 NULL 반환.
- response 반환. 에러 발생시 response free하고 NULL 반환.
---
    http_t *copy_http (http_t *http)

- 인자로 NULL 들어오면 에러 출력 NULL 반환.
- http_t *copy에 init_http_with_arg()로 http 각 요소 복사. NULL이면 에러 출력 NULL 반환.
- add_body_to_http() 실행해서 http의 body 요소 copy로 복사. -1이면 에러 출력 NULL 반환.
- for문으로 field_count만큼 field와 val을 add_field_to_http로 복사. -1이면 에러 출력 NULL 반환. 
- copy 반환. 에러 발생시 copy free하고 NULL 반환.
---
    void free_http (http_t *http)

- 인자 NULL이면 바로 NULL 반환.
- 모든 요소 free 후 http도 free.
---
    char *find_http_field_val (http_t *http, char *field);

- 인자로 받은 둘 중 하나라도 NULL이면 에러 출력 NULL 반환.
- for문으로 돌아가면 strcmp 동작, 0이면 해당 field값 반환.
- 다 돌고 없으면 NULL 반환.
---
    int add_field_to_http (http_t *http, char *field, char *val)

- 인자로 받은 값 하나라도 NULL이면 에러 출력 -1 반환.
- find_http_field_val(http, field)가 0 아니면 에러 출력 -1 반환.
- 현재 field_count + 1 이 max_field_count보다 크면 max_field_count 2배, http에 fields요소 realloc. 실패시 에러 출력 max_field_count / 2 하고 -1 반환.
- 성공하면 memset.
- 이후 copy_string(field)로 복사. 실패하면 에러 출력 -1 반환.
- 이후 copy_string(val)로 복사. 실패하면 에러 출력 -1 반환.
- field_count++ 하고 0 반환.
---
    int remove_field_from_http (http_t *http, char *field)

- 인자 중 하나라도 NULL이면 에러 출력 -1 반환.
- field_count 0이면 0 반환.
- int idx = -1;로 선언 초기화 하고 for문으로 돌면서 field 찾음. 찾으면 idx에 해당 index 넣고 break.
- idx -1이면 0 반환.
- idx번째 field, val free해줌.
- idx번째 이후 field, val 한 칸씩 앞으로 당기고 field_count--; 이후 0 반환.

    int add_body_to_http (http_t *http, size_t body_size, void *body_data)

- http NULL이면 에러 출력 -1 반환.
- body_size 0이거나 body_data NULL이면 0 반환.
- http의 body_data가 NULL이거나 body_size가 0이면 에러 출력 -1 반환.
- find_http_field_val(http, "Content-Length") == NULL이면 크기 32짜리 char content_length[32] 생성, sprintf로 body_size 값 저장
- add_field_to_http(http, "Content_Length", content_length) 실행, -1이면 오류 출력 -1 반환.
- body_size http에 저장, http의 body_data malloc으로 생성. NULL이면 오류 출력, 만약 remove_field_from_http(http, "Content-Length") -1이면 오류 출력, -1 반환
- memcpy로 http->body_data에 body_data를 body_size만큼 복사 0 반환.
---
    int remove_body_from_http (http_t *http)

- http NULL이면 오류 출력 -1 반환.
- http->body_data == NULL || http->body_size == 0 이면 0 반환.
- http->body_data free하고 http->body_data NULL로 연결 http_body_size 0으로 초기화.
- remove_field_from_http(http, "Content-Length") == -1 이면 에러 출력 -1 반환.
- 0 반환.
---
    ssize_t write_http_to_buffer (http_t *http, void** buffer_ptr)

- size_t buffer_size = 0 선언.
- http buffer_ptr 둘 중 하나라도 NULL이면 에러 출력 -1 반환.
- http->version이 NULL이면 에러 출력 -1 반환.
- buffer_size에 http->version 길이 + 1 만큼 더하고 method, path, status 각 요소 존재시 각 요소 길이 + 1 만큼 더함. 마지막에 + 1.
- for문으로 http->field_count만큼 buffer_size에 http->fields[i].field 길이 + 2 + http->fields[i].val 길이 + 2만큼 더함. for문 끝나고 + 2.
- http->body_size만큼 buffer_size 더함.
- *buffer_ptr NULL 아니면 *buffer_ptr free해줌.
- *buffer_ptr에 buffer_size만큼 calloc. 실패했으면 에러 출력 -1 반환.
- char *buffer에 *buffer_ptr 할당 후 buffer[0] = '\0'.
- 이후 http에 method 존재하면 내용을, 아니면 "" strcat하고 존재하면 " ", 아니면 "" strcat. path도 똑같이. 이후 version strcat 하고 " " strcat. status 존재하면 내용을, 아니면 "" strcat 하고 "\r\n" strcat. for문으로 http->field_count만큼 돌며 각 field cat, ": "cat, val cat, "\r\n" cat. 다 끝내고 "\r\n" cat. 마지막에 buffer += strlen(buffer) 해줌.
- 만약 *buffer_ptr + buffer_size가 http->body_size + buffer랑 다르면 에러 출력, *buffer_ptr free, NULL로 설정하거 -1 반환.
- 만약 http->body_data 존재하면 buffer에 http->body_data를 http->body_size만큼 복사하고 buffer += http->body_size 해줌.
- buffer_size 반환.

### B. Behavior
   1. View Album! 버튼을 클릭하면 web album에 12개의 이미지가 나온다. (초기 이미지)
   2. POST Image에 Browse버튼을 누르면 1MB 이하의 이름이 영어, 숫자로 된 .jpg 이미지를 업로드 할 수 있다.
   3. Go nowhere! 버튼을 누르면 404 Not Found 에러가 출력된다.
   4. Autheticate! 버튼을 누르고 username에 DCN, password에 FALL2023을 입력하면 secret image(초기값 : 교수님 사진)가 출력된다.

### C. Implementation Objectives
   서버는 아래 일들을 수행한다.
   1. Listening soket을 생성해 웹 브라우저의 연결을 받아들임
   2. HTTP 요청을 수신하고 구문을 분석함
   3. 요청에 따른 적절한 동작 수행
   4. 수행한 작업을 기반으로 HTTP 응답을 생성
   5. HTTP 응답을 웹 브라우저로 return
   2 ~ 5가 반복되며, http_engine.c 파일만 수정하면 된다.
   Details
   1. Listening soket 생성과 연결 수락
      - 서버가 시작될 때 main function이 http_engine.c 파일 내부의 server_engine() 함수를 부른다.
        TCP 소켓을 초기화하고 server_port argument에 묶는다.
      무한루프 내에서 보내지는 연결을 받아들이며 server_routine() 함수를 이용해 서버와 연결을 처리한다.
   2. HTTP 요청 받고 분석
      - HTTP 요청은 아래 사진을 따른다.
![University_homework/assets/146644182/48711af8-f30c-43d7-84bd-11ca1e97bce0](https://github.com/GeunSuYoon/University_homework/assets/146644182/48711af8-f30c-43d7-84bd-11ca1e97bce0)
      - sp는 ' ', cr는 '\r', 그리고 if는 '\n'
      - server_routine 함수 내에 header_buffer 문자 배열과 무한 루프가 있다.
        HTTP header를 1. 헤더 끝에 구분기호(i.e., \r\n\r\n)가 수신될 때, 2. 에러가 발생하거나 클라이언트가 연결을 해재할 때, 3. 클라이언트로 부터 너무 긴 헤더 메세지가 올 때 까지 루프 내에서 수신한다.
      - 헤더를 성공적으로 수신한 뒤, 클라이언트의 요청이 무엇인지 알기 위해 메세지를 분석한다. 받은 헤더 문자열로 부터 method, URL, 그리고 header field name & header field value tuple을 추출해야 한다.
      - 헤더 문자열을 http_t struct로 변환하기 위해 parse_http_header() 함수 구현을 추천한다. http_t struct는 http_util.c에서 제공하는 다양한 HTTP 요소를 관리하고 다루는 것을 도와주는 C 구조체다.
     다만, 필수적으로 해야하는 것은 아니다.
   3. 적절한 행동 취하기
      - 웹 브라우저에서 온 요청은 GET이나 POST method일 것이다.
        GET method는 HTML이나 CSS 파일과 같은 웹 페이지 요소를 웹 서버로부터 검색하기 위해 사용된다.
        POST method는 유저 이미지와 같은 클라이언트로부터 온 data를 HTTP 서버로 업로드하기 위해 사용된다.
      - GET method에서 HTTP 요청과 같은 URL로부터 요청된 파일을 return 해야한다.
      - POST method에서 HTTP 요청의 body에서 파일을 검색하고, 서버 내부에 파일로 저장하고, web album에 새롭게 올라온 파일을 보여줄 수 있도록 HTML 파일로 업로드해야한다.
      - 서버 기능을 지원할 것으로 예상되는 경우는 http_engine.c에서 주석화 되어 있으니 참고하자!
      - GET을 이용한 업로드는 이번 프로젝트에서 다루지 않는다.
   4. HTTP response 생성
      - 클라이언트 요청에 따라 해야할 행동을 받아 그에 따른 HTTP response를 웹 브라우저에 다시 보내야 할 것이다. http_t struct와 도움 함수들을 사용하자.
        response는 init_http_with_arg() 함수와 적절한 response 코드, response에 옳바른 header field 와 body를 붙여 초기화 해주어야 한다.
      - http_t struct와 HTTP response를 생성하는 함수의 예시는 "431 Header too large" 응답을 보낼 때 http_engine.c파일에 포함되어 있다.
   5. Returnning HTTP response
      - HTTP 응답이 준비되었을 때, 클라이언트에 response를 다시 보내야한다.
        The response massage는 비슷한 HTTP 요청 massage와 비슷한 구조지만, 요청 줄 대신 상태 줄을 사용한 HTTP protocol 표준 형태를 따라야 한다.
![University_homework/assets/146644182/1ded4fa8-cfa8-43e9-bd41-4a85ade01de5](https://github.com/GeunSuYoon/University_homework/assets/146644182/1ded4fa8-cfa8-43e9-bd41-4a85ade01de5)
      - 다행히 해당 부분은 이미 구현되어있다. http_t struct를 옳바른 HTTP response massage로 바꾸고 웹 브라우저로 전송한다. write_http_to_buffer() 함수를 읽어보면 동작원리가 이해가 갈 것이며, parse_http_header() 함수 구현에 도움이 될 것이다!

### D. 중요한 부분
   - 서버를 빌드하기 위해, make를 타이핑 해라. 코드를 바꾸고 난 뒤에는 다시 make 해줘야 한다. ctrl + c로 서버 프로그램을 닫자.
   - 함수의 return value, name 그리고 정의된 함수의 arguments는 바꾸면 안 된다!
   - 테스트 목적으로 다른 파일이나 함수를 정의하거나 만들 수 있지만, "http_engine.c" 파일만 제출해야 한다!
   - 필요하다면 http_engine.c 내부에 새로운 함수, 전역 변수, enum, struct등을 정의할 수 있다. 또한, C/POSIX 표준으로 정의된 다른 라이브러리를 사용할 수 있다. 설치가 필요한 외부 라이브러리 사용은 추천하지 않는다!
   - "http_engine.c" 내부 comments를 주의깊게 읽어라. http_t struct와 helper function을 잘 이용하면 HTTP 요소들의 관리와 조작이 쉬워질 것이다.
   - 또한, 웹 브라우저의 개발자 도구(Web Developer Tools) 또한 서버 디버깅에 도움을 줄 것이다. 네트워크 탭은 너의 서버와 웹 브라우저 사이 HTTP request 메시지와 HTTP response 메시지 교환을 보여줄 것이다!
   - 메모리 관리는 성적 반영 X. 하지만 프로젝트가 많은 양의 동적할당과 소켓과 파일 사이의 읽고 쓰는 것을 요구하므로 C 포인터와 메모리 구조를 이해하는 것은 좋다!
   - HTML, CSS 또는 JS와 같은 프론트 엔드 웹 페이지 요소들은 웹에 중요한 부분이다. 하지만, 이번 과제에서 요구하진 않는다!

## Aplication 2: BitTorrent-like P2P file sharing

### 스켈레톤 코드 목록

#### torrent.h
struct 목록 :

    typedef enum B_STAT
    {
        B_ERROR = -1,
        B_MISSING = 0,
        B_REQUESTED = 1,
        B_READY = 2
    } B_STAT;
---
    struct peer_data
    {
        torrent_t *torrent;
        char ip[STR_LEN];
        int port;
        size_t last_torrent_info_request_msec;
        size_t last_peer_list_request_msec;
        size_t last_block_status_request_msec;
        size_t last_block_request_msec;
        B_STAT *block_status;
    
    };
---
    struct torrent
    {
        torrent_engine_t *engine;
        HASH_t torrent_hash;
    
        char torrent_name[STR_LEN];
        size_t file_size;
        
        size_t num_blocks;
        HASH_t *block_hashes;
        B_STAT *block_status;
    
        size_t num_peers;
        size_t max_num_peers;
        peer_data_t **peers;
    
        size_t last_torrent_save_msec;
        size_t last_block_status_reset_msec;
    
        size_t download_speed;
        size_t download_speed_prev_num_blocks;
        size_t download_speed_prev_msec;
        void* data;
    
    };
--- 
    struct torrent_engine
    {
        int port;
        int listen_sock;
        HASH_t engine_hash;
    
        size_t num_torrents;
        size_t max_num_torrents;
        torrent_t **torrents;
    
        pthread_t thread;
        pthread_mutex_t mutex;
        int stop_engine;
    
    };
---
    typedef struct request_wrapper_data_t
    {
        peer_data_t *peer;
        torrent_t *torrent;
        size_t block_index;
    } request_wrapper_data_t;
---

#### torrent_utils.h
    HASH_t get_hash (void* data, size_t len);
- data의 hash value 얻어냄.
---
    HASH_t str_to_hash (char *str);
- str을 hash value로 바꿈
---
    int check_ipv4 (char *ip);
- ip가 ipv4인 확인함
---
    size_t get_time_msec();
- get 시간을 millisecond 단위로 확인함.
---
    size_t get_elapsed_msec();
- 시간 경과를 millisecond 단위로 확인함.
---
    int get_int_str_len (size_t num);
- str의 길이 num을 int로 변환함.
---
    ssize_t read_file (char *path, void *data);
- 메모리로 file을 읽음. 성공하면 읽은 bytes 수, 에러나면 -1 반환.
---
    ssize_t get_file_size (char *path);
- file의 크기를 확인함. 에러나면 -1 반환.
---
    ssize_t read_bytes (int socket, void *buffer, size_t size);
- size만큼 bytes를 socket에서 buffer로 읽음. 성공하면 읽은 bytes수, 에러나면 -1 반환.
---
    ssize_t write_bytes (int socket, void *buffer, size_t size);
- size만큼 bytes를 socket에서 buffer로 씀. 성공하면 읽은 bytes수, 에러나면 -1 반환.
---
    int kbhit();
- key press를 timeout 상관 없이 있는지 확인함. 있으면 1, 없으면 0 반환.
---


#### torrent_engine.h
    int request_torrent_info (peer_data_t *peer, torrent_t *torrent);
- peer나 torrent가 NULL이면 에러 출력 -2 반환
- peer의 last_torrent_info_request_msec 값을 get_elapsed_msec()으로 업데이트
- char msg[MSG_LEN] = {0} 선언 후 msg에 "REQUEST_TORRENT_INFO torrent->engine->engine_hash torrent->engine->port torrent->torrent_hash" 넣음.
- int peer_sock을 connect_socket(peer->ip, peer->port)로 초기화 하고, -1이면 에러 출력 -1 반환.
- -1 보다 작으면 에러 출력 -2 반환.
- 시간, 토렌트 해시, peer ip, port 출력.
- peer_sock에 메세지를 MSG_LEN만큼 write_bytes 하는데, 만약 적힌 bytes가 MSG_LEN보다 작으면 에러 출력 -2 반환.
- 0 반환.
---
    int push_torrent_info (peer_data_t *peer, torrent_t *torrent)
- 위 write_bytes부분까지 똑같게.
- peer_sock에 토렌트의 블록 해시, 블록 개수 * sizeof(HASH_t)를 write_bytes 해준 값이 토렌트의 블록 개수 * sizeof(HASH_t)랑 다르면 에러 출력 -2 반환.
- 0 반환.
---
    int handle_request_torrent_info (torrent_engine_t *engine, int peer_sock, peer_data_t *peer, torrent_t *torrent, char *msg_body);
- peer_sock이 0보다 작으면 에러 출력 -1 반환, engine, peer, torrent 중 하나라도 NULL이면 에러 출력 -1 반환.
- char *val을 msg_body를 " "로 strtok.
- val이 NULL이 아니면 에러 출력 -1 반환.
- 시간, 토렌트 해시, peer ip, port 출력.
- is_torrent_info_set으로 torrent 넣어서 1이거나 push_torrent_info로 peer에 torrent의 info push 실패하면 오류 출력 -1 반환.
- 0 반환.
---
    int handle_push_torrent_info (torrent_engine_t *engine, int peer_sock, peer_data_t *peer, torrent_t *torrent, char *msg_body);
- peer_sock이 0보다 작으면 에러 출력 -1 반환, engine, peer, torrent 중 하나라도 NULL이면 에러 출력 -1 반환.
- char *val을 msg_body를 " "로 strtok. char torrent_name[STR_LEN] = {0}으로 초기화 하고 strncpy로 val을 torrent_name에 복사.
- val을 한 번 더 strtok으로 " "까지 자르고 size_t file_size에 atoi로 숫자로 저장.
- val 또 짜르고 만일 NULL이 아니면 에러 출력 -1 반환.
- 시간, 토렌트 해시, peer ip, port 출력.
- is_torrent_info_set 으로 토렌트에 info가 있으면 에러 출력 -1 반환.
- set_torrent_info로 토렌트에 torrent_name, file_size 넣어서 0보다 작으면 에러 출력 -1 반환.
- peer_sock에 토렌트의 블록 해시, 블록 개수 * sizeof(HASH_t)를 read_bytes 한 값이 블록 개수 * sizeof(HASH_t)보다 작으면 에러 출력 -1 반환.
- 0 반환.
##### 이후로는 TODO
---
    int torrent_client (torrent_engine_t *engine);
- for문으로 size_t i가 0부터 engine의 토렌트 개수보다 작을 동안 돈다
    - engine의 i번째 토렌트를 torrent_t *torrent로 참조.
    - NULL이면 에러 출력 -1 반환.
    - TODO : is_torrent_info_set으로 토렌트 내부에 정보 있으면 get_elapsed_msec()과 토렌트의 마지막 토렌트 저장 시간을 빼서 TORRENT_SAVE_INTERVAL_MSEC보다 크면 에러 출력 -1 반환.
    - 만약 마지막 리셋 후 RESET_BLOCK_STATUS_INTERVAL_MSEC만큼 시간이 경과했으면, REQUESTED 상태의 블록을 MISSING 상태로 전환.
- for문으로 size_t peer_idx 0으로 선언 및 초기화. torrent->num_peers보다 작을 동안 돈다.
    - 해당 peer_idx번째 torrent의 peers를 peer_data_t *peer로 참조.
    - TODO : 만약 torrent의 info 받는 것에 실패했고 마지막 request로 부터 REQUEST_TORRENT_INFO_INTERVAL_MSEC만큼 경과했으면 peer로 부터 토렌트 정보 요청.
    - TODO : 만약 마지막 request로 부터 REQUEST_PEER_LIST_INTERVAL_MSEC만큼 경과했으면 peer의 list요청.
    - TODO : 만약 REQUEST_BLOCK_STATUS_INTERVAL_MSEC만큼 경과했으면 block status 요청.
    - TODO : 만약 REQUEST_BLOCK_INTERVAL_MSEC만큼 경과했으면 block 요청.
- 0 반환.
---
    int torrent_server (torrent_engine_t *engine)
- TODO : 들어오는 connection accept 하기.
- TODO : peer의 ip와 port 얻기.
- TODO : massage 읽기.
- TODO : massage에서 command parsing 하기
- TODO : massage에서 peer engine hash parsing 하기. 만약 local enging hash에 해당 hash가 존재하면 massage 무시.
- TODO : massage에서 peer listen port parsing 하기.
- TODO : massage에서 peer torrent hash parsing 하기.
- TODO : torrent가 peer에 없으면 peer에 torrent 더하기.
- TODO : massage commend에 맞춰 handler function 부르기.
  HINT : The handler function은 engine, peer_sock, peer, torrent, 그리고 msg_body가 필요함.
  torrent engine으로 engine, peer, torrent arguments 참조하기. massage를 보낸 peer와 massage가 요구하는 torrent.
  peer_sock argument는 accept_socket()의 반환을 참조.
  msg_body argument는 massage의 [TORRENT_HASH] 부분이 존재하면 참조.
- 0 반환.
---
    int listen_socket (int port)
- TODO : 소켓을 열고 들어오는 connections listen. 소켓 파일이 존재하면 해당 fd, 에러나면 -1 반환.
- 0 반환.
---
    int accept_socket(int listen_sock, struct sockaddr_in *cli_addr, socklen_t *clilen);
- TODO : 
- 0 반환.
---
    int connect_socket(char *server_ip, int port);

---
    int request_torrent_peer_list (peer_data_t *peer, torrent_t *torrent);

---
    int request_torrent_block_status (peer_data_t *peer, torrent_t *torrent);

---
    int request_torrent_block (peer_data_t *peer, torrent_t *torrent, size_t block_index);

---
    int push_torrent_peer_list (peer_data_t *peer, torrent_t *torrent);

---
    int push_torrent_block_status (peer_data_t *peer, torrent_t *torrent); 

---
    int push_torrent_block (peer_data_t *peer, torrent_t *torrent, size_t block_index);

---
    int handle_request_torrent_peer_list (torrent_engine_t *engine, int peer_sock, peer_data_t *peer, torrent_t *torrent, char *msg_body);

---
    int handle_request_torrent_block_status (torrent_engine_t *engine, int peer_sock, peer_data_t *peer, torrent_t *torrent, char *msg_body);

---
    int handle_request_torrent_block (torrent_engine_t *engine, int peer_sock, peer_data_t *peer, torrent_t *torrent, char *msg_body);

---
    int handle_push_torrent_peer_list (torrent_engine_t *engine, int peer_sock, peer_data_t *peer, torrent_t *torrent, char *msg_body);

---
    int handle_push_torrent_block_status (torrent_engine_t *engine, int peer_sock, peer_data_t *peer, torrent_t *torrent, char *msg_body);

---
    int handle_push_torrent_block (torrent_engine_t *engine, int peer_sock, peer_data_t *peer, torrent_t *torrent, char *msg_body);

---
### Introduction to Torrent Application
![University_homework/assets/146644182/28973d8a-0a3b-4bcc-8344-521d7270985f](https://github.com/GeunSuYoon/University_homework/assets/146644182/28973d8a-0a3b-4bcc-8344-521d7270985f)
   - 상기 사진에 있는 붉은색 요소들을 사용할 것이다!
   - 각 토렌트 어플들은 토렌트(토렌트 데이터베이스)의 목록, 클라이언트 함수, 서버 함수를 가지고 있다.
   - 각 토렌트는 파일을 32 킬로바이트의 블록으로 나눌 것이다. 클라리언트 함수는 토렌트 내부의 데이터베이스를 되풀이하고, 토렌트 블록을 다운하기 위해 요구되는 행동(해당 블록에 있는 것으로 알려져있는 peer에 블록 데이터를 요구)을 수행할 것이다.
   - 서버 함수는 원격 peer의 메세지를 받고 연결의 명령에 따라 적절한 조치를 취할 것이다(HTTP 서버가 하는 것 처럼). A request message, a push massge 이라는 두 다른 타입의 메세지를 서버가 받을 것이다.
   - A request massage는 원격 peer의 클라이언트 함수로부터 보내지고 특정 토렌트 요소를 요구할 것이다. 원격 peer가 요구하는 토렌트 요소는 네 가지가 있다. A request massage를 받은 후, 서버 함수는 적절한 handler를 불러 일치하는 push massage가 요구하는 요소를 반환할 것이다.
      1. 토렌트 정보(파일 이름, 사이즈 등)
      2. 해당 토렌트에 알려진 peer 목록
      3. Peer의 최근 다운로드 상태
      4. 블록 데이터
   - A push massage는 원격 peer의 서버 함수로부터 보내지고 클라이언트 함수가 요구하는 토렌트 요소르 가지고 있다. 원격 peer가 push할 수 있는 토렌트 요소는 네 가지가 있다. A push massage가 도착했을 때, 서버 함수는 해당 massage에 적절한 handler를 부르고 pushed element를 저장하기 위해 토렌트 데이터베이스에 업데이트 할 것이다.
   - 우리 어플리케이션은 hash_value에 따라 토렌트를 관리할 것이다. 토렌트가 파일로부터 생성됐을 때, 유일한 hash_value를 받게 될 것이다. 그 파일을 다운로드받길 원하는 유저는 hash_value를 이용해 그 토렌트를 추가할 수 있다. Peer가 토렌트를 추가했을 때, 업데이트가 필요한 토렌트 요소를 위한 reqeust massage가 자동적으로 보내질 것이다!
   - 따라서, 우리 토렌트 어플리케이션 내 massage 교환은 아래의 순서를 따른다.
     1. 토렌트 클라이언트는 토렌트가 필요로하는 원격 peer로 부터 업데이트되는 특정 토렌트 요소 찾고, 해당 요소에 대한 request massage를 원격 peer에게 보낸다.
     2. 원격 peer의 서버 클라이언트는 request massage를 받고, request massage를 다루고, 특정 요구되는 토렌트 요소를 포함한 일치하는 push massage를 응답한다.
     3. 요청한 peer의 서버 클라이언트는 원격 peer로부터 push massage를 받고, pushed torrent element를 저장하기 위해 토렌트 데이터베이스를 업데이트한다.
   - TA binary를 포함해 작동 시킬 수 있고, 아래 단계를 밟으면 너의 어플리케이션의 결과가 기대했던 방법으로 동작했는지 알 수 있다.
     1. 포함된 binary 파일에 실행 권한을 준다   chmod +x ./torrent*
     2. 너의 시스템에 맞는 binary를 실행한다   ./torrent_<YOUR_SYSTEM> 62123
        help를 이용해 보조 커맨드를 볼 수 있다.
        status를 입력해 데이터 베이스 내 토렌트와 다운로드 상태를 볼 수 있다.
        info <IDX>를 입력해 토렌트의 인덱스 <IDX>번째 정보를 자세히 볼 수 있다.
        hash에 기반한 토렌트를 추가하기 위해, + 버튼을 이용해 VSC의 다른 터미널을 열고 다른 port number의 토렌트 피어를 동작시켜라.
        add <HASH>를 입력해 hash에 기반한 토렌트를 추가할 수 있다. status 커맨드를 실행시키면 토렌트에 아무런 정보가 없다고 반환할 것이다.
        add_peer <IDX> <IP> <PORT>를 이용해 peer를 추가해라. The original peer를 추가하는 것은 어플리케이션이 자동적으로 토렌트 정보를 회수하고 다운로드를 시작하게 만들 것이다.
        watch <IDX>로 자동으로 정리된 토렌트 정보를 볼 수 있다. 또한 인덱스를 제거한 watch 로 자동으로 정리된 데이터베이스 상태를 볼 수 있다. watch mode를 탈출하기 위해, 엔터를 눌러라.
