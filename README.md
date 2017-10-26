# Android 와 Node.js 통신으로 간단한 Login 처리

### 설명
____________________________________________________

![SignIn](https://github.com/Hooooong/DAY30_HttpSignin/blob/master/image/Signin.gif)

- Android 와 Node.js 통신으로 간단한 Login 처리

- Server 파일 : [database.js](https://github.com/Hooooong/DAY30_Nodejs-POST-DB-/blob/master/database.js)

### KeyPoint
____________________________________________________

- Node.js 설정

  - 참조 : [Node.js, MongoDB 설정](https://github.com/Hooooong/DAY30_Nodejs-POST-DB-)

- Server 실행

  1. MongoDB 를 실행한다.

      - 명령어 : `mongod --dbpath '설정한 data 경로'`

      ![MongoDB실행](https://github.com/Hooooong/DAY30_HttpSignin/blob/master/image/MongoDB.PNG)

  2. MongoDB Terminal 실행

      - 명령어 : `mongodb`

      - 사용하는 DB와 User 가 있는지 검색

      ![MongoDB Terminal](https://github.com/Hooooong/DAY30_HttpSignin/blob/master/image/User.PNG)

  3. Server 실행

      - 명령어 : `node database`

- Android 통신

  - 서버에 JSON String 으로 보내고, 받을 때 JSON String 을 객체로 변환

  ```java
  Sign sign = new Sign();
  sign.setId(mEmail);
  sign.setPw(mPassword);
  // 객체를 JSON String 으로 변환
  String json = new Gson().toJson(sign);

  // json 데이터로 보내고, json 데이터로 받는다.
  String result = Remote.sendPost(json, address);

  // JSON String 을 객체로 변환
  Result rst= new Gson().fromJson(result, Result.class);
  ```

  - 참조 : [HTTP통신](https://github.com/Hooooong/DAY25_HTTPConnect#httpconnection)

### Code Review
____________________________________________________

- LoginActivity.java

  - Server 와 통신하는 구간

  - Address 설정을 해주고 AsyncTask 를 통해 서버와 HTTP 통신을 한다.

  ```java
  public class LoginActivity extends AppCompatActivity implements LoaderCallbacks<Cursor> {

      // 주소 설정
      private final String address = "http://172.30.102.91:8090/signin";

      // 생략...........,

      /**
       * Represents an asynchronous login/registration task used to authenticate
       * the user.
       */
      public class UserLoginTask extends AsyncTask<Void, Void, String> {

          private final String mEmail;
          private final String mPassword;

          UserLoginTask(String email, String password) {
              mEmail = email;
              mPassword = password;
          }

          @Override
          protected String doInBackground(Void... params) {
              // TODO: attempt authentication against a network service.

              Sign sign = new Sign();
              sign.setId(mEmail);
              sign.setPw(mPassword);
              // 객체를 JSON String 으로 변환
              String json = new Gson().toJson(sign);

              // JSON String으로 보내고, JSON String 으로 받는다.
              String result = Remote.sendPost(json, address);
              // TODO: register the new account here.
              return result;
          }

          @Override
          protected void onPostExecute(String result) {
              mAuthTask = null;
              showProgress(false);

              // JSON String 을 객체로 변환
              Result rst= new Gson().fromJson(result, Result.class);

              if (rst.isOk()) {
                  Toast.makeText(LoginActivity.this, "WELCOME!!", Toast.LENGTH_SHORT).show();
              } else {
                  mPasswordView.setError(getString(R.string.error_incorrect_password));
                  mPasswordView.requestFocus();
              }
          }

          @Override
          protected void onCancelled() {
              mAuthTask = null;
              showProgress(false);
          }
      }
  }
  ```

- Remote.java

  - httpconnection 설정 클래스

  ```java
  public class Remote {

      public static String sendPost(String jsonData, String urlString) {
          String result = "";

          try {
              Log.e("Remote", urlString);
              URL url = new URL(urlString);
              HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();
              httpURLConnection.setRequestMethod("POST");

              // 보낼 데이터가 있다는 것을 알려준다.
              httpURLConnection.setDoOutput(true);
              // 데이터를 보낸다.
              OutputStream os = httpURLConnection.getOutputStream();
              os.write(jsonData.getBytes());
              os.close();
              if (httpURLConnection.getResponseCode() == HttpURLConnection.HTTP_OK) {
                  // 응답이 성공적으로 받은 것....
                  InputStreamReader isr = new InputStreamReader(httpURLConnection.getInputStream());
                  BufferedReader br = new BufferedReader(isr);
                  String temp = "";
                  if ((temp = br.readLine()) != null) {
                      result += temp ;
                  }

                  br.close();
                  isr.close();
              }else{
                  // 응답이 성공적으로 받지 못한 것...
                  result += "ERROR";
                  Log.e("ServerError", httpURLConnection.getResponseCode() + " , " + httpURLConnection.getResponseMessage());
              }
          } catch (Exception e) {
              Log.e("Error", e.toString());
          }
          return result;
      }

      // 생략
  }
  ```

- Result.java

  - 결과값을 받는 클래스

  ```java
  public class Result {

      public static final String OK = "200";

      private String code;
      private String msg;

      public String getCode() {
          return code;
      }

      public void setCode(String code) {
          this.code = code;
      }

      public String getMsg() {
          return msg;
      }

      public void setMsg(String msg) {
          this.msg = msg;
      }

      public boolean isOk() {
          return OK.equals(code);
      }
  }
  ```

- Sign.java

  - 입력값을 받는 클래스

  ```java
  public class Sign {
      private String id;
      private String pw;

      public String getId() {
          return id;
      }

      public void setId(String id) {
          this.id = id;
      }

      public String getPw() {
          return pw;
      }

      public void setPw(String pw) {
          this.pw = pw;
      }
  }
  ```
