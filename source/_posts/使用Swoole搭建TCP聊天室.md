---
title: 使用Swoole搭建TCP聊天室
date: 2018-10-16 11:33:21
tags:
    - PHP
categories:
    - PHP
    - Swoole
    - TCP
comments: true
---

```php
// Server.php

namespace Chat;

use Chat\proto\Body;
use Chat\proto\ChatRequest;
use Chat\proto\Info;
use Chat\proto\RequestType;
use Swoole\Table;
<!-- more -->
class Server
{
    private $swooleServer;

    public function __construct($ip = '127.0.0.1', $port = 9501)
    {
        $this->swooleServer = new \Swoole\Server($ip, $port);
        $this->swooleServer->set([
            'worker_num' => 2,
            'daemonize' => false,
        ]);
        $this->swooleServer->connectTable = $this->connectTable();
        $this->swooleServer->onlineTable = $this->onlineTable();
    }

    public function start()
    {
        $this->swooleServer->on('connect', [$this, 'onConnect']);
        $this->swooleServer->on('receive', [$this, 'onReceive']);
        $this->swooleServer->on('close', [$this, 'onClose']);
        $this->swooleServer->start();
    }


    public function onConnect(\Swoole\Server $serv, $fd)
    {
        /** @var Table $connect */
        $this->swooleServer->connectTable->set($fd, [
            'id' => $fd,
            'name' => 'default',
            'login_time' => time(),
            'is_login' => 0,
        ]);
        /** @var Table $online */
        print_r("Current online count: {$this->swooleServer->onlineTable->count()}");

    }

    public function onReceive(\Swoole\Server $serv, $fd, $fromId, $data)
    {
        if (!trim($data)) {
            return;
        }
        if (!$this->isJson($data)) {
            $serv->send($fd, "incorrect data format: $data\n");
            return;
        }
        print_r("Receive $fd data: $data \n");
        $request = new ChatRequest();
        $request->mergeFromJsonString($data);
        switch ($request->getType()) {
            case RequestType::RequestType_LOGIN:
                $this->login($request->getInfo(), $fd);
                break;
            case RequestType::RequestType_MESSAGE:
                $this->sendMessage($request->getBody(), $serv);
                break;
            default:
                break;
        }
    }

    public function onClose(\Swoole\Server $serv, $fd)
    {
        /** @var Table $connect */
        $connect = $this->swooleServer->connectTable;
        $loginInfo = $connect->get($fd);
        $connect->del($fd);
        if ($loginInfo['is_login']) {
            /** @var Table $online */
            $online = $this->swooleServer->onlineTable;
            $online->del($loginInfo['name']);
        }
    }

    private function login(Info $info, $fd): bool
    {
        if (!$info->getUsername()) {
            return false;
        }
        print_r("Logging in user: {$info->getUsername()}\n");
        $userInfo = [
            'id' => uniqid(),
            'name' => $info->getUsername(),
            'login_time' => time(),
            'client_id' => $fd,
        ];
        return $this->swooleServer->onlineTable->set($info->getUsername(), $userInfo);
    }

    private function sendMessage(Body $body, \Swoole\Server $serv): bool
    {
        /** @var Table $online */
        $online = $this->swooleServer->onlineTable;
        if (!$online->exist($body->getTo())) {
            print_r("User {$body->getTo()} is not exist\n");
            return false;
        }
        $info = $online->get($body->getTo());
        print_r("Send to user info: " . json_encode($info) . "\n");
        return $serv->send($info['client_id'], $body->getMsg());
    }

    private function isJson(string $data): bool
    {
        json_decode($data);
        if (json_last_error() !== JSON_ERROR_NONE) {
            return false;
        }

        return true;
    }

    private function connectTable(): Table
    {
        $table = new \Swoole\Table(1024);
        $table->column('id', Table::TYPE_INT);
        $table->column('name', Table::TYPE_STRING, 64);
        $table->column('login_time', Table::TYPE_INT);
        $table->column('is_login', Table::TYPE_INT);
        $table->create();

        return $table;
    }

    private function onlineTable(): Table
    {
        $table = new Table(1024);
        $table->column('id', Table::TYPE_INT);
        $table->column('name', Table::TYPE_STRING, 64);
        $table->column('login_time', Table::TYPE_INT);
        $table->column('client_id', Table::TYPE_INT);
        $table->create();

        return $table;
    }
}
```


```php
// Client.php

namespace Chat;

use Chat\proto\Body;
use Chat\proto\ChatRequest;
use Chat\proto\Info;
use Chat\proto\RequestType;
use Swoole\Process;

class Client
{
    private $swooleClient;

    public function __construct($ip = '127.0.0.1', $port = 9501)
    {
        $this->swooleClient = new \Swoole\Client(SWOOLE_SOCK_TCP);
        $this->swooleClient->connect($ip, $port, 10);
        $this->login();
    }

    public function start()
    {
        echo "请输入要发送消息的好友名称: ";
        $username = $this->input();

        $process = new Process(function(Process $worker) use ($username) {
            while (true) {
                $msg = $this->input();
                if ($msg) {
                    $this->sendMessage($username, $msg);
                }
            }
        });
        $process->start();

        while (true) {
            $msg = @$this->swooleClient->recv();
            if (trim($msg)) {
                echo date('Y-m-d H:i:s', time()), " {$msg}\n";
            }
        }
        $this->close();
    }

    private function login()
    {
        echo "请输入用户名: ";
        $username = $this->input();
        $login = new ChatRequest();
        $login->setType(RequestType::RequestType_LOGIN);
        $info = new Info();
        $info->setUsername($username);
        $login->setInfo($info);
        $login->setSendingTime(time());

        $this->swooleClient->send($login->serializeToJsonString());
    }

    private function sendMessage($to, $msg): bool
    {
        $request = new ChatRequest();
        $request->setType(RequestType::RequestType_MESSAGE);
        $body = new Body();
        $body->setTo($to);
        $body->setMsg($msg);
        $request->setBody($body);
        $request->setSendingTime(time());
        $this->swooleClient->send($request->serializeToJsonString());

        return true;
    }

    private function input(): string
    {
        return trim(fgets(STDIN));
    }

    private function close()
    {
        $this->swooleClient->close();
    }
}

```

```php
// ClientAsync.php

namespace Chat;

use Chat\proto\Body;
use Chat\proto\ChatRequest;
use Chat\proto\Info;
use Chat\proto\RequestType;
use Swoole\Process;

class ClientAsync
{
    private $client;
    private $host;
    private $port;

    public function __construct($host = '127.0.0.1', $port = 9501)
    {
        $this->host = $host;
        $this->port = $port;
        $this->client = new \Swoole\Client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_ASYNC);
        swoole_set_process_name('client-async:master');
        $this->signal();
    }

    public function start()
    {
        $this->client->on('connect', [$this, 'onConnect']);
        $this->client->on('receive', [$this, 'onReceive']);
        $this->client->on('close', [$this, 'onClose']);
        $this->client->on('error', [$this, 'onError']);
        $this->client->connect($this->host, $this->port);
    }

    public function onConnect(\Swoole\Client $client)
    {
        $this->login();

        do {
            echo "请输入要发送消息的好友名称: ";
        } while (!$username = $this->input());

        $process = new Process(function(Process $worker) use ($username) {
            swoole_set_process_name('client-async:worker');
            while (true) {
                $msg = $this->input();
                if ($msg) {
                    $this->sendMessage($username, $msg);
                }
            }
        });
        $process->start();
        $GLOBALS['child'] = $process;

        $currentPid = posix_getpid();
        $this->output("Master process pid(1): {$currentPid}\n");
        $this->output("Worker process pid(2): {$process->pid}\n");
    }

    public function onReceive(\Swoole\Client $client, $data)
    {
        echo date('Y-m-d H:i:s', time()), " {$data}\n";
    }

    public function onClose(\Swoole\Client $client)
    {
        $this->output('Client close');
    }

    public function onError(\Swoole\Client $client)
    {
        $this->output('Swoole client error: ' . $client->errCode);
    }

    private function login()
    {
        do {
            echo "请输入用户名: ";
        } while (!$username = $this->input());

        $login = new ChatRequest();
        $login->setType(RequestType::RequestType_LOGIN);
        $info = new Info();
        $info->setUsername($username);
        $login->setInfo($info);
        $login->setSendingTime(time());

        $this->client->send($login->serializeToJsonString());
    }

    private function sendMessage($to, $msg): bool
    {
        $request = new ChatRequest();
        $request->setType(RequestType::RequestType_MESSAGE);
        $body = new Body();
        $body->setTo($to);
        $body->setMsg($msg);
        $request->setBody($body);
        $request->setSendingTime(time());
        $this->client->send($request->serializeToJsonString());

        return true;
    }

    private function input(): string
    {
        return trim(fgets(STDIN));
    }

    private function output($msg)
    {
        echo $this->now(), ' ', $msg;
    }

    private function now(): string
    {
        return date('Y-m-d H:i:s', time());
    }

    private function signal()
    {
        Process::signal(SIGTERM, function($signal) {
            global $child;
            Process::kill($child->pid);
            Process::wait();
            exit();
        });
    }
}
```
