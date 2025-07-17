# 스레드를 활용한 채팅 프로그램 구현

Server
```java


public class Server {
    private static Set<ClientHandler> clients = ConcurrentHashMap.newKeySet();
    private static int clientCount = 0;

    public static void main(String[] args) {
        try (ServerSocket serverSocket = new ServerSocket(12345)) {
            System.out.println("서버 대기 중 ... " );

            // 서버 콘솔 입력도 별도 스레드로 처리
            new Thread(() -> {
                Scanner sc = new Scanner(System.in);
                while(true) {
                    String msg = sc.nextLine();
                    broadcast("[서버] " + msg);
                }
            }).start();

            while (true) {
                Socket socket = serverSocket.accept();
                clientCount++;
                ClientHandler handler = new ClientHandler(socket, "Client-" + clientCount);
                clients.add(handler);
                new Thread(handler).start();
            }

        } catch (IOException e ) {
            e.printStackTrace();
        }
    }

    public static void broadcast(String message) {
        for (ClientHandler client: clients) {
            client.send(message);
        }
    }

    static class ClientHandler implements Runnable {

        private Socket socket;
        private String name;
        private PrintWriter out;
        private BufferedReader in;

        public ClientHandler(Socket socket, String name) {
            this.socket = socket;
            this.name = name;
        }

        public void send(String message) {
            if (out != null) {
                out.println(message);
            }
        }

        @Override
        public void run() {
            System.out.println(name + " 연결됨 : " + socket.getRemoteSocketAddress());

            try {
                in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                out = new PrintWriter(socket.getOutputStream(), true);
                out.println("[알림] 당신의 이름은 " + name + " 입니다.");

                String message;

                while((message = in.readLine()) != null) {
                    System.out.println(name + " : " + message);
                    broadcast("[" + name + "]" + message);
                }
            } catch (IOException e) {
                System.out.println(name + "연결 종료");
            } finally {
                try {
                    socket.close();
                } catch (IOException ignored) {
                    clients.remove(this);
                }
            }
        }
    }
}
```

Client
```java
public class Client {
    public static void main(String[] args) {
        try (Socket socket = new Socket("localhost", 12345)) {
            System.out.println("서버에 연결됨.");

            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true) ;

            // 수신 스레드
            new Thread(() -> {
                try {
                    String line;
                    while((line = in.readLine()) != null) {
                        System.out.println("[서버] " + line);
                    }
                } catch (IOException e) {
                    System.out.println("서버 연결 종료됨. ");
                }
            }).start();

            // 쓰기 스레드
            Scanner scanner = new Scanner(System.in);
            while(true) {
                String msg = scanner.nextLine();
                System.out.println("나 : " + msg);
                out.println(msg);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```
