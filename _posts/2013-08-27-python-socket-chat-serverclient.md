---
title: 'Python Socket Chat Server&#038;Client'
author: Rodumani
layout: post
permalink: /2013/08/python-socket-chat-serverclient/
categories:
  - Python
tags:
  - Python
  - Socket
  - TCP
---
Network Programming 공부의 일환으로 Socket 실습으로 작성했던 Multi-client Chat server & client 이다.

귓말 기능은 없고, 로컬 접속 + 전체채팅만 가능하고, Client에서는 &#8220;:exit&#8221;이라는 명령어가 있어서 채팅에서 나갈 수 있다. 

<pre class="lang:python decode:true " title="Chat Server" >#!/usr/bin/python
#-*- coding:utf-8 -*-

from socket import *
import thread,time
import sys
import signal

HOST = 'localhost'
BUF_SIZE=1024

MAX_ID = 0
client_list = []
serversock = None

def get_id() : 
    global MAX_ID 
    MAX_ID+=1 
    return MAX_ID

class Client() :
    def __init__(self,conn,addr) :
        self.conn = conn
        self.addr = addr
        name = get_id()
        self.name = str(name)

def signal_handler(signal,frame) :
    global client_list
    if serversock!=None :
        serversock.shutdown(SHUT_RDWR)
        for i in client_list :
            i.conn.shutdown(SHUT_RDWR)
            i.conn.close()
        serversock.close()
        print "Socket Closed"
    print "Close the server"
    sys.exit(0)

def main() :
    global HOST,BUF_SIZE,client_list,serversock
    signal.signal(signal.SIGINT,signal_handler)
    if len(sys.argv) == 2 :
        PORT = int( sys.argv[1] )
        serversock = socket(AF_INET,SOCK_STREAM)
        serversock.bind((HOST,PORT))
        serversock.listen(1)
        try :
            while True:
                conn,addr = serversock.accept()
                client = Client(conn,addr)
                client_list.append(client)
                print "Client Connected : " + str(client.name)
                thread.start_new_thread(handle_client,(client,))
        except Exception,e:
            print e
            serversock.close()
    else :
        print "need port number"



def handle_client(client):
    global client_list
    try :
        conn = client.conn
        addr = client.addr
        broadcast(client,"=== New Client Connected : "+str(client.name)+ " ===")
        while True:
            recv = conn.recv(BUF_SIZE).strip()
            recv_len = 0
            if not recv:
                print "Connection Closed"+ str(addr)
                broadcast(client,"=== Client Disconnected : "+str(client.name) + "===" )
                client_list.remove(client)
                return 
            else :
                recv_len = int(recv)
            recv_data = conn.recv(recv_len)
            broadcast(client,str(client.name)+" : " +recv_data)
    except Exception,e:
        print e
        conn.close()

def broadcast(me,msg) :
    global client_list
    bad_list = []
    for client_iter in client_list :
        try:
            if client_iter == me : 
                continue
            client_iter.conn.send("BROADCAST")
            client_iter.conn.send(msg)
            print "Send messge : ",msg,client_iter.addr
            back = client_iter

        except Exception,e:
            bad_list.append(client_iter)
            print e
            continue
    for i in bad_list :
        i.conn.close()
        client_list.remove(i) 
</pre>

<pre class="lang:python decode:true " title="Chat Client" >#!/usr/bin/python
#-*- coding:utf-8 -*-

from socket import *
import thread,time
import sys
import os
import signal

HOST = 'localhost'
BUF_SIZE = 1024

clientsock = None

sock_lock = thread.allocate_lock()

def signal_handler(signal,frame) :
    if clientsock != None :
        clientsock.shutdown(SHUT_RDWR)
        clientsock.close()
        print "Socket Closed"
    print "Close the client"
    os._exit(0)

def main() :
    global HOST,BUF_SIZE,clientsock
    signal.signal(signal.SIGINT,signal_handler)
    if len(sys.argv) == 2 : 
        PORT = int(sys.argv[1]) 

        clientsock = socket(AF_INET,SOCK_STREAM)
        clientsock.connect((HOST,PORT))
    
        thread.start_new_thread(handle_server_msg,(clientsock,))

        while True:
            mesg = raw_input("")
            if mesg != "" :
                if mesg.find(":exit") == 0:
                    break
                sock_lock.acquire()
                clientsock.send(str(len(mesg)))
                clientsock.send(mesg)
                sock_lock.release()      
        clientsock.shutdown(SHUT_RDWR)
        clientsock.close()
    else :
        print "need port number"


def handle_server_msg(sock) :
    try: 
        while True: 
            msg = sock.recv(BUF_SIZE) 
            if not msg : 
                raise Exception
            if msg == "BROADCAST" :
                sock_lock.acquire()
                msg = sock.recv(BUF_SIZE)
                print msg
                sock_lock.release()

    except Exception,e:
        sock.close()
        print "Server Shutdownded. Client is going to closed"
        os._exit(0)

if __name__ == "__main__": 
    main()
</pre>