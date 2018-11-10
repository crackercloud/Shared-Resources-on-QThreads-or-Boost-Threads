<h1 id="shared-resources-on-qthreads-or-boost-threads">Shared Resources on QThreads or Boost Threads</h1>
<p>This repository describes how to manage shared variables with two or more QThreads or Boost Threads</p>
<p>To put it simply, the purpose of threads is to allow code to run in parallel. You have to manage threads if you want to use same variables at the same time with two or more threads in multi-threaded applications. It doesn&rsquo;t matter whether threads are QThread or Boost Thread. We&rsquo;ll do both of them.</p>
<p>Examples have been made with QtCreator.</p>
<p>Let&rsquo;s write a simple code that includes four QThread threads and some arithmetic operations made on the same variable.</p>
<ol>
<li>Create a project with Qt Console Application</li>
<li>Crate a class that inherits from QThread. In my example, the class name is  &ldquo;mythread&rdquo;.</li>
</ol>
<p>Here is the code header file contains:</p>
<pre><code>#ifndef MYTHREAD_H
#define MYTHREAD_H
#include &lt;QtCore&gt;
#include &lt;QDebug&gt;

class mythread : public QThread
{
public:
    mythread(QString name, int *number);
        void run();
        void method1();
        void method2();

private:
        int *threadNum;
        QString threadName;
};

#endif // MYTHREAD_H
</code></pre>
<p>I have created two public void methods.the void run() method is necessary for QThread start.</p>
<p>Here is the code mythread.cpp file contains:</p>
<pre><code>#include "mythread.h"

mythread::mythread(QString name, int * number)
{
    threadNum = number;
    threadName = name;
}

void mythread::run()
{
    method1();
    method2();
}

void mythread::method1()
{
    *threadNum += 2;
    qDebug() &lt;&lt; "I'm in " &lt;&lt; threadName &lt;&lt; " with threadNum = " &lt;&lt; *threadNum;
    *threadNum *= 2;
    qDebug() &lt;&lt; "I'm in " &lt;&lt; threadName &lt;&lt; " with threadNum = " &lt;&lt; *threadNum;
}

void mythread::method2()
{
    *threadNum *= 3;
    qDebug() &lt;&lt; "I'm in " &lt;&lt; threadName &lt;&lt; " with threadNum = " &lt;&lt; *threadNum;
    *threadNum /= 4;
    qDebug() &lt;&lt; "I'm in " &lt;&lt; threadName &lt;&lt; " with threadNum = " &lt;&lt; *threadNum;
}
</code></pre>
<p>method1() first adds  2 to given number and then multiplies by 4. method2() first mutiplies the given number by 5 and then divides by 3. These methods are called in QThread run() method.</p>
<p>Here is the code main.cpp file contains:</p>
<pre><code>#include &lt;QCoreApplication&gt;
#include "mythread.h"

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    int number = 8;

    mythread m1Thread("thread1", &amp;number);
    mythread m2Thread("thread2", &amp;number);
    mythread m3Thread("thread3", &amp;number);
    mythread m4Thread("thread4", &amp;number);

    m1Thread.start();
    m2Thread.start();
    m3Thread.start();
    m4Thread.start();

    a.exec();

    return 0;
}
</code></pre>
<p>In the main.cpp file, I created four different threads with different names in order. Each thread uses the same number by reference.</p>
<p>When I run the code, the output has occurred as follows:</p>
<blockquote>
<p>//  thread2 called method1()<br />
I&rsquo;m in  &ldquo;thread2&rdquo;  with threadNum =  10         // number is 10</p>
<p>// thread1 called method2()<br />
  // and see the thread1 used the variable. thread2 did not finish process yet!!!<br />
I&rsquo;m in  &ldquo;thread1&rdquo;  with threadNum =  14<br />
I&rsquo;m in  &ldquo;thread3&rdquo;  with threadNum =  12<br />
I&rsquo;m in  &ldquo;thread3&rdquo;  with threadNum =  64<br />
I&rsquo;m in  &ldquo;thread1&rdquo;  with threadNum =  32<br />
I&rsquo;m in  &ldquo;thread3&rdquo;  with threadNum =  384<br />
I&rsquo;m in  &ldquo;thread3&rdquo;  with threadNum =  96<br />
I&rsquo;m in  &ldquo;thread1&rdquo;  with threadNum =  288<br />
I&rsquo;m in  &ldquo;thread2&rdquo;  with threadNum =  128<br />
I&rsquo;m in  &ldquo;thread2&rdquo;  with threadNum =  864<br />
I&rsquo;m in  &ldquo;thread1&rdquo;  with threadNum =  216<br />
I&rsquo;m in  &ldquo;thread2&rdquo;  with threadNum =  54<br />
I&rsquo;m in  &ldquo;thread4&rdquo;  with threadNum =  16<br />
I&rsquo;m in  &ldquo;thread4&rdquo;  with threadNum =  108<br />
I&rsquo;m in  &ldquo;thread4&rdquo;  with threadNum =  324<br />
I&rsquo;m in  &ldquo;thread4&rdquo;  with threadNum =  81</p>
</blockquote>
<p><strong>As seen above, before one of the threads finished its process on variable, one of the other threads took an action on the same variable. This is something that should not be!!!!</strong></p>
<p>The access to the same variable of different threads must be in order! If we want to do this correctly, we need to use <strong>QMutex.</strong><br />
The QMutex class provides access serialization between threads.</p>
<p>The <strong>QMutex</strong> class provides access serialization between threads.</p>
<p>If we redesign our code with QMutex;</p>
<pre><code>#ifndef MYTHREAD_H
#define MYTHREAD_H
#include &lt;QtCore&gt;

class mythread : public QThread
{
public:
    mythread(QString name,QMutex *mutex, int * number);
    void run();
    void method1();
    void method2();

private:
    int *threadNum;
    QString threadName;
    QMutex *mtx;
};

#endif // MYTHREAD_H
</code></pre>
<p>We define a QMutex member varible as pointer. <strong>We have to use same QMutex for lock and unlock operations.</strong></p>
<p>Here is the code mythread.cpp file contains:</p>
<pre><code>#include "mythread.h"
#include &lt;QtCore&gt;
#include &lt;QDebug&gt;

mythread::mythread(QString name, QMutex *mutex, int * number)
{
    threadNum = number;
    threadName = name;
    mtx = mutex;
}

void mythread::run()
{
    method1();
    method2();
}

void mythread::method1()
{
    mtx-&gt;lock();

    *threadNum += 2;
    qDebug() &lt;&lt; "I'm im " &lt;&lt; threadName &lt;&lt; " with threadNum = " &lt;&lt; *threadNum;
    *threadNum *= 2;
    qDebug() &lt;&lt; "I'm im " &lt;&lt; threadName &lt;&lt; " with threadNum = " &lt;&lt; *threadNum;

    mtx-&gt;unlock();
}

void mythread::method2()
{
    mtx-&gt;lock();

    *threadNum *= 3;
    qDebug() &lt;&lt; "I'm im " &lt;&lt; threadName &lt;&lt; " with threadNum = " &lt;&lt; *threadNum;
    *threadNum /= 4;
    qDebug() &lt;&lt; "I'm im " &lt;&lt; threadName &lt;&lt; " with threadNum = " &lt;&lt; *threadNum;

    mtx-&gt;unlock();
}
</code></pre>
<p>So, here is the code main.cpp file contains:</p>
<pre><code>#include &lt;QCoreApplication&gt;
#include "mythread.h"

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    int number = 8;
    QMutex mutex;

    mythread m1Thread("thread1", &amp;mutex, &amp;number);
    mythread m2Thread("thread2", &amp;mutex, &amp;number);
    mythread m3Thread("thread3", &amp;mutex, &amp;number);
    mythread m4Thread("thread4", &amp;mutex, &amp;number);

    m1Thread.start();
    m3Thread.start();
    m2Thread.start();
    m4Thread.start();

    a.exec();

    return 0;
}
</code></pre>
<p><sub><em>When you call lock() in a thread, other threads that try to call lock() in the same place will block until the thread that got the lock calls unlock().</em></sub></p>
<p>We passed the QMutex variable by reference to threads. As you can see above on the mythread.cpp, we locked before doing process with &ldquo;number&rdquo; variable and unlocked at the end of methods. Thus, only one thread&rsquo;s method(method1 or method2) can modify &ldquo;number&rdquo; at any given time. In this way, the result will be correct!! </p>
<p>Let&rsquo;s examine our result:</p>
<blockquote>
<p>I&rsquo;m im  &ldquo;thread3&rdquo;  with threadNum =  10<br />
I&rsquo;m im  &ldquo;thread3&rdquo;  with threadNum =  20<br />
I&rsquo;m im  &ldquo;thread3&rdquo;  with threadNum =  60<br />
I&rsquo;m im  &ldquo;thread3&rdquo;  with threadNum =  15<br />
I&rsquo;m im  &ldquo;thread1&rdquo;  with threadNum =  17<br />
I&rsquo;m im  &ldquo;thread1&rdquo;  with threadNum =  34<br />
I&rsquo;m im  &ldquo;thread1&rdquo;  with threadNum =  102<br />
I&rsquo;m im  &ldquo;thread1&rdquo;  with threadNum =  25<br />
I&rsquo;m im  &ldquo;thread2&rdquo;  with threadNum =  27<br />
I&rsquo;m im  &ldquo;thread2&rdquo;  with threadNum =  54<br />
I&rsquo;m im  &ldquo;thread2&rdquo;  with threadNum =  162<br />
I&rsquo;m im  &ldquo;thread2&rdquo;  with threadNum =  40<br />
I&rsquo;m im  &ldquo;thread4&rdquo;  with threadNum =  42<br />
I&rsquo;m im  &ldquo;thread4&rdquo;  with threadNum =  84<br />
I&rsquo;m im  &ldquo;thread4&rdquo;  with threadNum =  252<br />
I&rsquo;m im  &ldquo;thread4&rdquo;  with threadNum =  63</p>
</blockquote>
<p>IF we put the 1s sleep between methods in run() function we can see better results:</p>
<pre><code>#include "mythread.h"
#include &lt;QtCore&gt;
#include &lt;QDebug&gt;

mythread::mythread(QString name, QMutex *mutex, int * number)
{
    threadNum = number;
    threadName = name;
mtx = mutex;
}

void mythread::run()
{
    method1();
    sleep(1);
    method2();
}

void mythread::method1()
{
    mtx-&gt;lock();

    *threadNum += 2;
    qDebug() &lt;&lt; "I'm im " &lt;&lt; threadName &lt;&lt; " method1() with threadNum = " &lt;&lt; *threadNum;
    *threadNum *= 2;
    qDebug() &lt;&lt; "I'm im " &lt;&lt; threadName &lt;&lt; " method1() with threadNum = " &lt;&lt; *threadNum;

    mtx-&gt;unlock();
}

void mythread::method2()
{
    mtx-&gt;lock();

    *threadNum *= 3;
    qDebug() &lt;&lt; "I'm im " &lt;&lt; threadName &lt;&lt; " method2() with threadNum = " &lt;&lt; *threadNum;
    *threadNum /= 4;
    qDebug() &lt;&lt; "I'm im " &lt;&lt; threadName &lt;&lt; " method2() with threadNum = " &lt;&lt; *threadNum;

    mtx-&gt;unlock();
}
</code></pre>
<p>Here is the new results:</p>
<blockquote>
<p>I&rsquo;m im  &ldquo;thread3&rdquo;  method1() with threadNum =  10<br />
I&rsquo;m im  &ldquo;thread3&rdquo;  method1() with threadNum =  20<br />
I&rsquo;m im  &ldquo;thread1&rdquo;  method1() with threadNum =  22<br />
I&rsquo;m im  &ldquo;thread1&rdquo;  method1() with threadNum =  44<br />
I&rsquo;m im  &ldquo;thread2&rdquo;  method1() with threadNum =  46<br />
I&rsquo;m im  &ldquo;thread2&rdquo;  method1() with threadNum =  92<br />
I&rsquo;m im  &ldquo;thread4&rdquo;  method1() with threadNum =  94<br />
I&rsquo;m im  &ldquo;thread4&rdquo;  method1() with threadNum =  188<br />
I&rsquo;m im  &ldquo;thread3&rdquo;  method2() with threadNum =  564<br />
I&rsquo;m im  &ldquo;thread3&rdquo;  method2() with threadNum =  141<br />
I&rsquo;m im  &ldquo;thread1&rdquo;  method2() with threadNum =  423<br />
I&rsquo;m im  &ldquo;thread1&rdquo;  method2() with threadNum =  105<br />
I&rsquo;m im  &ldquo;thread2&rdquo;  method2() with threadNum =  315<br />
I&rsquo;m im  &ldquo;thread2&rdquo;  method2() with threadNum =  78<br />
I&rsquo;m im  &ldquo;thread4&rdquo;  method2() with threadNum =  234<br />
I&rsquo;m im  &ldquo;thread4&rdquo;  method2() with threadNum =  58</p>
</blockquote>
<p>As you can see above, there is  no conflict between the threads anymore. This is the right solution to manage threads that use shared variables between them.</p>
<hr />
<h4 id="of-course-we-can-do-that-with-boost-thread">Of course we can do that with <strong>Boost Thread!</strong></h4>
<p>It&rsquo;s important to implement boost library path and include path to .pro file. In our case, boost is installed under /opt/ path.</p>
<p>Here is the .pro file that we used:</p>
<pre><code>QT -= gui

CONFIG += c++11 console
CONFIG -= app_bundle

#The following define makes your compiler emit warnings if you use
#any feature of Qt which as been marked deprecated (the exact warnings
#depend on your compiler). Please consult the documentation of the
#deprecated API in order to know how to port your code away from it.
DEFINES += QT_DEPRECATED_WARNINGS

#You can also make your code fail to compile if you use deprecated APIs.
#In order to do so, uncomment the following line.
#You can also select to disable deprecated APIs only up to a certain version of Qt.
#DEFINES += QT_DISABLE_DEPRECATED_BEFORE=0x060000    # disables all the APIs deprecated before Qt 6.0.0

SOURCES += \
        main.cpp \
    mythread.cpp

#Default rules for deployment.
qnx: target.path = /tmp/$${TARGET}/bin
else: unix:!android: target.path = /opt/$${TARGET}/bin
!isEmpty(target.path): INSTALLS += target

HEADERS += \
    mythread.h

LIBS += -L"/opt/boost_1_54_0/stage/lib"\
        -lboost_system\
        -lboost_thread\
        -lboost_chrono\
        -lboost_filesystem

INCLUDEPATH += /opt/boost_1_54_0
DEPENDPATH += /opt/boost_1_54_0
</code></pre>
<p>Here is the mytread.h file:</p>
<pre><code>#ifndef MYTHREAD_H
#define MYTHREAD_H
#include &lt;QtCore&gt;
#include &lt;QDebug&gt;
#include &lt;boost/thread.hpp&gt;

class mythread : public QObject
{
    Q_OBJECT
public:
    mythread(QString name, boost::mutex *mutex,int * number);
    ~mythread();
    void Init();
    void method1();
    void method2();
    void threadExec();

private:
    int *threadNum;
    QString threadName;
    boost::thread threadVar;
    boost::mutex *mtx;
};

#endif // MYTHREAD_H
</code></pre>
<p>Here is mythread.cpp file:</p>
<pre><code>#include "mythread.h"
#include &lt;boost/thread/thread.hpp&gt;

mythread::mythread(QString name, boost::mutex *mutex,int * number)
{
    threadNum = number;
    threadName = name;
    mtx = mutex;
}

mythread::~mythread()
{
}

void mythread::Init()
{
    threadVar = boost::thread(boost::bind(&amp;mythread::threadExec, this));
}

void mythread::threadExec()
{
    method1();
    sleep(1);
    method2();
}

void mythread::method1()
{
    mtx-&gt;lock();

    *threadNum += 2;
    qDebug() &lt;&lt; "I'm im " &lt;&lt; threadName &lt;&lt; " method1() with threadNum = " &lt;&lt; *threadNum;
    *threadNum *= 2;
    qDebug() &lt;&lt; "I'm im " &lt;&lt; threadName &lt;&lt; " method1() with threadNum = " &lt;&lt; *threadNum;

    mtx-&gt;unlock();
}

void mythread::method2()
{
    mtx-&gt;lock();

    *threadNum *= 3;
    qDebug() &lt;&lt; "I'm im " &lt;&lt; threadName &lt;&lt; " method2() with threadNum = " &lt;&lt; *threadNum;
    *threadNum /= 4;
    qDebug() &lt;&lt; "I'm im " &lt;&lt; threadName &lt;&lt; " method2() with threadNum = " &lt;&lt; *threadNum;

    mtx-&gt;unlock();
}
</code></pre>
<p>Here is the new results:</p>
<blockquote>
<p>I&rsquo;m im  &ldquo;thread1&rdquo;  method1() with threadNum =  10<br />
I&rsquo;m im  &ldquo;thread1&rdquo;  method1() with threadNum =  20<br />
I&rsquo;m im  &ldquo;thread2&rdquo;  method1() with threadNum =  22<br />
I&rsquo;m im  &ldquo;thread2&rdquo;  method1() with threadNum =  44<br />
I&rsquo;m im  &ldquo;thread3&rdquo;  method1() with threadNum =  46<br />
I&rsquo;m im  &ldquo;thread3&rdquo;  method1() with threadNum =  92<br />
I&rsquo;m im  &ldquo;thread4&rdquo;  method1() with threadNum =  94<br />
I&rsquo;m im  &ldquo;thread4&rdquo;  method1() with threadNum =  188<br />
I&rsquo;m im  &ldquo;thread1&rdquo;  method2() with threadNum =  564<br />
I&rsquo;m im  &ldquo;thread1&rdquo;  method2() with threadNum =  141<br />
I&rsquo;m im  &ldquo;thread3&rdquo;  method2() with threadNum =  423<br />
I&rsquo;m im  &ldquo;thread3&rdquo;  method2() with threadNum =  105<br />
I&rsquo;m im  &ldquo;thread2&rdquo;  method2() with threadNum =  315<br />
I&rsquo;m im  &ldquo;thread2&rdquo;  method2() with threadNum =  78<br />
I&rsquo;m im  &ldquo;thread4&rdquo;  method2() with threadNum =  234<br />
I&rsquo;m im  &ldquo;thread4&rdquo;  method2() with threadNum =  58</p>
</blockquote>
<p>We got the same result with Boost Thread as we got before with QThread. </p>
<p>I hope my experience will be useful for all.</p>
<p><em><br />
Ömer BULUT<br />
Elecrical &amp; Electronics Engineer<br />
Software Developer</em></p>
