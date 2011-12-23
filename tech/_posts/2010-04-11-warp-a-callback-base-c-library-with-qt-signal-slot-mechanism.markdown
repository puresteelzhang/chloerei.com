---
layout: post
title: warp a callback-base C library with qt signal-slot mechanism
---

<a href="http://chloerei.com/wp-content/uploads/2010/04/logo.png"><img class="size-full wp-image-139 alignright" title="logo" src="http://chloerei.com/wp-content/uploads/2010/04/logo.png" alt="" width="59" height="71" /></a>Sometime we need to call some funtion from C library, etc. libpcap, libnids. But these C library is base on callback mechanism, how to let it work with qt signal-slot mechanism?

After some search, I found a way to wrap a callback-base C library using QThread and C++ singleton class. (<a href="http://www.qtcentre.org/threads/26371-emit-a-signal-from-inside-a-callback-routine?highlight=callback">refer to a page on qtcentre.org</a>)

for example, wrap libnids. <a href="http://libnids.sourceforge.net/">Libnids</a> is an implementation of an E-component of Network Intrusion Detection System. It use callback to pass network packet info.

To wrap this library using Qt signal, there are some points:

<!--more-->
<h2>1. Include C head file</h2>
When include a C head file in a C++ source, must extern it to "C"

File: nidsthread.cpp
<pre lang="cpp-qt" line="3" escaped="true">extern "C"{
#include "nids.h"
}</pre>
<h2>2. Call C funtion in a new thread</h2>
If don't call funtion from libnids in a new thread, it will block the main thread.

Qt provide thread by QThread, so I create a class named NidsThread inherit QThread.

File: nidsthread.h
<pre lang="cpp-qt" line="7" escaped="true">class NidsThread : public QThread
{
    Q_OBJECT
public:
    NidsThread();
    // init libnids in run(), it will execute in a new thread
    void run();

private:
    proxy pro;
    // callback funtion must be static
    static void callback(struct tcp_stream *a_tcp, void ** this_time_not_needed);
};
</pre>
As if all nids funtion must be call in static class funtion, etc. init(), the callback funtion, exit(). (Actually I don't know why, I hope someone tell me. : P)

Then call nids_init() in NidsThread::run, and register callback funtion.

File: nidsthread.cpp
<pre lang="cpp-qt" line="7" escaped="true">void NidsThread::run()
{
    if (!nids_init()) {
        qDebug(nids_errbuf);
    }
    nids_register_tcp((void*)(callback));
    nids_run();
}

void NidsThread::callback(struct tcp_stream *a_tcp, void ** this_time_not_needed)
{
    // use Proxy to emit a signal for this callback funtion
    Proxy::singleton()-&gt;callback();
}
</pre>
The proxy class will be create later.
<h2>3. Create a Proxy class to emit signals</h2>
We can call nids funtion in a new Thread now, but qt can't emit a signal without a object. So we need to create a proxy to emit signal.

At this example, I define a tcpCallback signal in the Proxy class, and a callback funtion to emit it.
<pre lang="cpp-qt" line="10" escaped="true">class proxy : public QThread
{
    Q_OBJECT
public:
    Proxy();
    // just use to emit a signal
    void callback(struct tcp_stream *a_tcp){ emit tcpCallback(a_tcp);}
    static Proxy* singleton();
signals:
    void tcpCallback(struct tcp_stream *a_tcp);
private:
    static Proxy *m_singleton;
};
</pre>
And make it singleton
<pre lang="cpp-qt" line="1" escaped="true">#include "proxy.h"

Proxy * Proxy::m_singleton = new Proxy();

Proxy::Proxy()
{
}

Proxy* Proxy::singleton()
{
    return m_singleton;
}
</pre>
Now we can get the proxy singleton by calling Proxy::singleton(), and connect it's signal, witch will emit in NidsThread callback funtion.

Let the proxy singleton will reduce some code for pass object between two thread. (If know a  better method, tell me)
<h2>4. Connect signal</h2>
<pre lang="cpp-qt" line="15" escaped="true">    MainWindow w;

    NidsThread nid;
    QObject::connect(Proxy::singleton(), SIGNAL(ipCallback()), &amp;w, SLOT(parseCallback()));
    nid.start();
</pre>
At this example I didn't pass the struct tcp_stream through signal. As if signal can only deliver Type registered in QMetaType. registered struct tcp_stream or pick-up the useful data from the struct and change to QMetaType ... I don't keep on. : P

a exmaple source. dependent Libnids
<a href="http://chloerei.com/wp-content/uploads/2010/04/nidsproject.tar.gz">nidsproject.tar</a>
