
## 리펙토링 코드 
``` python
import sys
import time
from PyQt5.QtWidgets import (
    QApplication, QWidget, QPushButton, QVBoxLayout, QLabel, QLineEdit, QProgressBar
)
from PyQt5.QtCore import Qt, QThread, pyqtSignal
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import NoAlertPresentException


class WorkerThread(QThread):
    progress_changed = pyqtSignal(int)

   # 매 0.03초마다 1씩 증가 (총 0~100까지 101번 emit, 전체 약 3초 소요)
    def run(self):
        for i in range(101):
            time.sleep(0.03)
            self.progress_changed.emit(i)


class MyApp(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("PyQt5 GUI 예제")
        self.resize(500, 300)
        self.driver = None  # 브라우저 객체 초기화

        # 입력창
        self.id = QLineEdit()
        self.id.setPlaceholderText("무신사 아이디를 입력하세요")

        self.pwd = QLineEdit()
        self.pwd.setPlaceholderText("비밀번호를 입력하세요")
        self.pwd.setEchoMode(QLineEdit.Password)

        # 결과 출력 라벨
        self.result_label = QLabel("결과가 여기에 표시됩니다.")
        self.label = QLabel("진행 상태가 여기에 표시됩니다.")

        # 프로그래스 바
        self.progress_bar = QProgressBar()
        self.progress_bar.setAlignment(Qt.AlignCenter)
        self.progress_bar.setValue(0)

        # 버튼
        self.loginButton = QPushButton('무신사 로그인')
        self.startButton = QPushButton('시작')
        self.endButton = QPushButton('종료')
        self.move_to_mypage = QPushButton('마이페이지 이동')

        self.loginButton.clicked.connect(self.musinsa_auto)
        self.startButton.clicked.connect(self.start_progress)
        self.endButton.clicked.connect(self.program_dead)
        self.move_to_mypage.clicked.connect(self.go_myPage)



        # 레이아웃 설정
        layout = QVBoxLayout()
        layout.addWidget(self.id)
        layout.addWidget(self.pwd)
        layout.addWidget(self.result_label)
        layout.addWidget(self.loginButton)
        layout.addWidget(self.startButton)
        layout.addWidget(self.endButton)
        layout.addWidget(self.progress_bar)
        layout.addWidget(self.label)
        layout.addWidget(self.move_to_mypage)

        self.setLayout(layout)

    def start_progress(self):
        self.label.setText("작업 진행 중 ...")
        self.thread = WorkerThread()
        self.thread.progress_changed.connect(self.update_progress)
        self.thread.start()

    def update_progress(self, value):
        self.progress_bar.setValue(value)
        if value == 100:
            self.label.setText("작업 완료!")

    def program_dead(self):
        if self.driver:
            self.driver.quit()
        QApplication.quit()

    def go_to_raffle(self):
        self.driver.get("https://www.musinsa.com/preuser/main?gf=A")


    def go_myPage(self):
        self.driver.get("https://www.musinsa.com/mypage")


    def get_in_site(self):
        self.driver = webdriver.Chrome()
        self.driver.get("https://www.musinsa.com/auth/login?referer=https%3A%2F%2Fwww.musinsa.com%2Fmypage")
        return self.driver

    def musinsa_auto(self):
        self.driver = self.get_in_site()
        id_value = self.id.text()
        pwd_value = self.pwd.text()
        self.auto_login(self.driver, id_value, pwd_value)
 

    def auto_login(self, driver, id_text, pwd_text):
        try:
            # ID와 비밀번호 입력
            driver.find_element(By.XPATH, '//input[@title="아이디 입력"]').send_keys(id_text)
            driver.find_element(By.XPATH, '//input[@title="비밀번호 입력"]').send_keys(pwd_text)

            # 로그인 버튼 클릭
            driver.find_element(By.XPATH, '//button[contains(text(), "로그인")]').click()

           


            WebDriverWait(driver, 5).until(
                lambda d: "mypage" in d.current_url or "auth/login" in d.current_url
            )

            if "mypage" in driver.current_url:
                self.result_label.setText("로그인 성공!")

                try:
                    popup_button = WebDriverWait(self.driver, 3).until(
                        EC.element_to_be_clickable((By.XPATH, '//button[contains(text(), "등급 혜택 확인하기")]'))
                    )
                    popup_button.click()  # 클릭하면 모달이 사라지는 경우
                except:
                    pass  # 모달이 없으면 무시

            else :
                self.result_label.setText("로그인 실패 . 아이디 / 비밀번호를 확인하세요. ")

        except Exception as e:
            self.result_label.setText(f"오류 발생: {e}")



if __name__ == '__main__':
    app = QApplication(sys.argv)
    window = MyApp()
    window.show()
    sys.exit(app.exec_())
```

## 리펙토링 코드  
``` python 
import sys
import time
from PyQt5.QtWidgets import (
    QApplication, QWidget, QPushButton, QVBoxLayout, QLabel, QLineEdit, QProgressBar
)
from PyQt5.QtCore import Qt, QThread, pyqtSignal
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import NoAlertPresentException, TimeoutException


class WorkerThread(QThread):
    progress_changed = pyqtSignal(int)

    def run(self):
        for i in range(101):
            time.sleep(0.03)
            self.progress_changed.emit(i)


class MyApp(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("무신사 자동 로그인")
        self.resize(500, 300)
        self.driver = None

        self.setup_ui()
        self.connect_signals()

    def setup_ui(self):
        self.id = QLineEdit()
        self.id.setPlaceholderText("무신사 아이디를 입력하세요")

        self.pwd = QLineEdit()
        self.pwd.setPlaceholderText("비밀번호를 입력하세요")
        self.pwd.setEchoMode(QLineEdit.Password)

        self.result_label = QLabel("결과가 여기에 표시됩니다.")
        self.status_label = QLabel("진행 상태가 여기에 표시됩니다.")

        self.progress_bar = QProgressBar()
        self.progress_bar.setAlignment(Qt.AlignCenter)

        self.login_btn = QPushButton("무신사 로그인")
        self.start_btn = QPushButton("진행 바 실행")
        self.mypage_btn = QPushButton("마이페이지 이동")
        self.quit_btn = QPushButton("종료")

        layout = QVBoxLayout()
        layout.addWidget(self.id)
        layout.addWidget(self.pwd)
        layout.addWidget(self.result_label)
        layout.addWidget(self.login_btn)
        layout.addWidget(self.start_btn)
        layout.addWidget(self.quit_btn)
        layout.addWidget(self.progress_bar)
        layout.addWidget(self.status_label)
        layout.addWidget(self.mypage_btn)
        self.setLayout(layout)

    def connect_signals(self):
        self.login_btn.clicked.connect(self.login)
        self.start_btn.clicked.connect(self.run_progress_bar)
        self.quit_btn.clicked.connect(self.quit_app)
        self.mypage_btn.clicked.connect(lambda: self.navigate_to("https://www.musinsa.com/mypage"))

    def run_progress_bar(self):
        self.status_label.setText("작업 진행 중...")
        self.thread = WorkerThread()
        self.thread.progress_changed.connect(self.update_progress)
        self.thread.start()

    def update_progress(self, value):
        self.progress_bar.setValue(value)
        if value == 100:
            self.status_label.setText("작업 완료!")

    def quit_app(self):
        if self.driver:
            self.driver.quit()
        QApplication.quit()

    def navigate_to(self, url):
        if self.driver:
            self.driver.get(url)

    def login(self):
        self.driver = webdriver.Chrome()
        self.driver.get("https://www.musinsa.com/auth/login?referer=https%3A%2F%2Fwww.musinsa.com%2Fmypage")

        try:
            WebDriverWait(self.driver, 10).until(EC.presence_of_element_located((By.TAG_NAME, "body")))
            self.enter_credentials()
            self.handle_popup_if_present()
        except TimeoutException:
            self.result_label.setText("페이지 로딩 실패")

    def enter_credentials(self):
        try:
            self.driver.find_element(By.XPATH, '//input[@title="아이디 입력"]').send_keys(self.id.text())
            self.driver.find_element(By.XPATH, '//input[@title="비밀번호 입력"]').send_keys(self.pwd.text())
            self.driver.find_element(By.XPATH, '//button[contains(text(), "로그인")]').click()

            WebDriverWait(self.driver, 5).until(
                lambda d: "mypage" in d.current_url or "auth/login" in d.current_url
            )

            if "mypage" in self.driver.current_url:
                self.result_label.setText("✅ 로그인 성공!")
            else:
                self.result_label.setText("❌ 로그인 실패: 아이디/비밀번호 확인")
        except Exception as e:
            self.result_label.setText(f"❗ 로그인 중 오류: {e}")

    def handle_popup_if_present(self):
        try:
            popup_btn = WebDriverWait(self.driver, 3).until(
                EC.element_to_be_clickable((By.XPATH, '//button[contains(text(), "등급 혜택 확인하기")]'))
            )
            popup_btn.click()
        except TimeoutException:
            pass  # 팝업이 없으면 그냥 통과


if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = MyApp()
    window.show()
    sys.exit(app.exec_())

```
