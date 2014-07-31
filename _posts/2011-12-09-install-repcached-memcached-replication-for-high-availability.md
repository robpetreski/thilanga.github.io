---
layout: post
title: Install Repcached (Memcached Replication) For High-Availability
date: 2011-12-09
comments: true
---

When you have a dynamic website which handles lots of user queries, as a Web Master my top priorities are to keep the site
up and running with minimum downtime (I meant 0 downtime) and to keep the site healthy to response back to users in very very short time.

So, keeping those 2 main tasks in my mind, I was able to track down the problem which was haunting for sometimes.

It was non optimized queries which ran through our [WSO2 Developer Portal](http://wso2.org/ "WSO2 Developer Portal").
Due to this issue, portal's MySQL load was always high. So the answer was to reduce the MySQL load.

I used [Memcached](http://memcached.org/ "Memcached") to minimize database load. Memcached increases the performance and scalability
of dynamic MySQL-driven websites by caching data and objects in memory.


Setting up Memcached is fairly simple. You can Install using APT (Debian based) or [download the tar and compile it on the server]

After Installing Memcached with MySQL, it gave a good performance boost to our Developer Portal until a node's cache got expired.

I noticed that some users couldn't upload / attach files to [OR Forum](http://wso2.org/forum) or new article.
After some series of testing and debugging sessions, I was able confirm that we had a problem in Memcached while accessing in a cluster enviroment.

The reason was : Lets' say when users are accessing the site in a peak time, first request is severed from node1.
Then the second request gets routed to node3 or Node4 ([backup nodes] (http://www.pitigala.org/2011/12/drupal-scaling-and-performance-tuning.html))
due to high load in the cluster, User Drupal can not access cache objects created during the first request.
Because of that, user receives lots of unexpected results.

Then I Installed [RepCached](http://repcached.lab.klab.org/) to support Replication in Memcached.
Repcached helps to keep redundancy memcached system and that was the solution I was looking for.


####Installing RepCached

- Download the Latest version of repcached from [http://repcached.lab.klab.org/](http://repcached.lab.klab.org/)
- Install some extra packages on Debian (apt-get install libevent-dev g++ make)
- Install repcached from tar
- Extract files (tar xvf memcached-1.2.8-repcached-2.2.tar)
- go to the directory (cd memcached-1.2.8-repcached-2.2/)
- Enable replication before install (./configure --enable-replication)
- Install (make && make install)

####Configure repcache

- create the file (vim /etc/memcachedrep)
- Create the init Script (vim /etc/init.d/memcachedrep)
- chmod +x /etc/init.d/memcacherep
- update-rc.d memcachedrep defaults

>Copy / Past code1 to step 1
```bash
```

                <div></div>
                <div># -x &lt; ip_addr &gt; hostname or IP address of the master replication server</div>
                <div># -X &lt; num &gt; TCP port number of the master (default: 11212)</div>
                <div>DAEMON_ARGS="-m 128 -p 11211 -u root -P /var/run/memcachedrep.pid -d -x 10.100.1.10</div>
            </div>
        </div>
        <div><br/></div>
        <div><u>Copy / Past code2 to step 2</u></div>
        <div>
            <div>#! /bin/sh</div>
            <div>### BEGIN INIT INFO</div>
            <div># Provides:             memcached</div>
            <div># Required-Start:       $syslog</div>
            <div># Required-Stop:        $syslog</div>
            <div># Should-Start:         $local_fs</div>
            <div># Should-Stop:          $local_fs</div>
            <div># Default-Start:        2 3 4 5</div>
            <div># Default-Stop:         0 1 6</div>
            <div># Short-Description:    memcached - Memory caching daemon replicated</div>
            <div># Description:          memcached - Memory caching daemon replicated</div>
            <div>### END INIT INFO</div>
            <div># Author: Michael </div>
            <div>#</div>
            <div># Please remove the "Author" lines above and replace them</div>
            <div># with your own name if you copy and modify this script.</div>
            <div># Do NOT "set -e"</div>
            <div># PATH should only include /usr/* if it runs after the mountnfs.sh script</div>
            <div>PATH=/sbin:/usr/sbin:/bin:/usr/bin</div>
            <div>DESC="memcachedrep"</div>
            <div>NAME=memcached</div>
            <div>DAEMON=/usr/local/bin/$NAME</div>
            <div>DAEMON_ARGS="--options args"</div>
            <div>PIDFILE=/var/run/memcachedrep.pid</div>
            <div>SCRIPTNAME=/etc/init.d/$DESC</div>
            <div>VERBOSE="yes"</div>
            <div># Exit if the package is not installed</div>
            <div>[ -x "$DAEMON" ] || exit 0</div>
            <div># Read configuration variable file if it is present</div>
            <div>[ -r /etc/$DESC ] &amp;&amp; . /etc/$DESC</div>
            <div># Load the VERBOSE setting and other rcS variables</div>
            <div>. /lib/init/vars.sh</div>
            <div># Define LSB log_* functions.</div>
            <div># Depend on lsb-base (&gt;= 3.0-6) to ensure that this file is present.</div>
            <div>. /lib/lsb/init-functions</div>
            <div>#</div>
            <div># Function that starts the daemon/service</div>
            <div>#</div>
            <div>do_start()</div>
            <div>{</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># Return</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#   0 if daemon has been started
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#   1 if daemon was already running
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#   2 if daemon could not be
                started
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>start-stop-daemon --start --quiet
                --pidfile $PIDFILE --exec $DAEMON --test &gt; /dev/null \
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;">  </span>|| return 1</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>start-stop-daemon --start --quiet
                --pidfile $PIDFILE --exec $DAEMON -- \
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;">  </span>$DAEMON_ARGS \</div>
            <div><span class="Apple-tab-span" style="white-space: pre;">  </span>|| return 2</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># Add code here, if necessary, that
                waits for the process to be ready
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># to handle requests from services
                started subsequently which depend
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># on this one.  As a last resort,
                sleep for some time.
            </div>
            <div>}</div>
            <div>#</div>
            <div># Function that stops the daemon/service</div>
            <div>#</div>
            <div>do_stop()</div>
            <div>{</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># Return</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#   0 if daemon has been stopped
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#   1 if daemon was already stopped
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#   2 if daemon could not be
                stopped
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#   other if a failure occurred
            </div>
            <div>    start-stop-daemon --stop --quiet --retry=TERM/30/KILL/5 --pidfile $PIDFILE --name $NAME
            </div>
            <div>    RETVAL="$?"</div>
            <div>    [ "$RETVAL" = 2 ] &amp;&amp; return 2</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># Wait for children to finish too if
                this is a daemon that forks
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># and if the daemon is only ever run
                from this initscript.
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># If the above conditions are not
                satisfied then add some other code
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># that waits for the process to drop all
                resources that could be
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># needed by services started
                subsequently.  A last resort is to
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># sleep for some time.</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>start-stop-daemon --stop --quiet
                --oknodo --retry=0/30/KILL/5 --exec $DAEMON
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>[ "$?" = 2 ] &amp;&amp; return 2</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># Many daemons don't delete their
                pidfiles when they exit.
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>rm -f $PIDFILE</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>return "$RETVAL"</div>
            <div>}</div>
            <div>#</div>
            <div># Function that sends a SIGHUP to the daemon/service</div>
            <div>#</div>
            <div>do_reload() {</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># If the daemon can reload its
                configuration without
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># restarting (for example, when it is
                sent a SIGHUP),
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># then implement that here.</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>start-stop-daemon --stop --signal 1
                --quiet --pidfile $PIDFILE --name $NAME
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>return 0</div>
            <div>}</div>
            <div>case "$1" in</div>
            <div>  start)</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>[ "$VERBOSE" != no ] &amp;&amp;
                log_daemon_msg "Starting $DESC" "$NAME"
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>do_start</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>case "$?" in</div>
            <div><span class="Apple-tab-span" style="white-space: pre;">  </span>0|1) [ "$VERBOSE" != no ] &amp;&amp;
                log_end_msg 0 ;;
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;">  </span>2) [ "$VERBOSE" != no ] &amp;&amp;
                log_end_msg 1 ;;
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>esac</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>;;</div>
            <div>  stop)</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>[ "$VERBOSE" != no ] &amp;&amp;
                log_daemon_msg "Stopping $DESC" "$NAME"
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>do_stop</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>case "$?" in</div>
            <div><span class="Apple-tab-span" style="white-space: pre;">  </span>0|1) [ "$VERBOSE" != no ] &amp;&amp;
                log_end_msg 0 ;;
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;">  </span>2) [ "$VERBOSE" != no ] &amp;&amp;
                log_end_msg 1 ;;
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>esac</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>;;</div>
            <div>  #reload|force-reload)</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># If do_reload() is not implemented then
                leave this commented out
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># and leave 'force-reload' as an alias
                for 'restart'.
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#log_daemon_msg "Reloading $DESC"
                "$NAME"
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#do_reload</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#log_end_msg $?</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#;;</div>
            <div>  restart|force-reload)</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># If the "reload" option is implemented
                then remove the
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span># 'force-reload' alias</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>log_daemon_msg "Restarting $DESC"
                "$NAME"
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>do_stop</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>case "$?" in</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>  0|1)</div>
            <div><span class="Apple-tab-span" style="white-space: pre;">  </span>do_start</div>
            <div><span class="Apple-tab-span" style="white-space: pre;">  </span>case "$?" in</div>
            <div><span class="Apple-tab-span" style="white-space: pre;">   </span>0) log_end_msg 0 ;;</div>
            <div><span class="Apple-tab-span" style="white-space: pre;">   </span>1) log_end_msg 1 ;; # Old process is
                still running
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;">   </span>*) log_end_msg 1 ;; # Failed to start
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;">  </span>esac</div>
            <div><span class="Apple-tab-span" style="white-space: pre;">  </span>;;</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>  *)</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>  <span class="Apple-tab-span"
                                                                                             style="white-space: pre;"> </span>#
                Failed to stop
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;">  </span>log_end_msg 1</div>
            <div><span class="Apple-tab-span" style="white-space: pre;">  </span>;;</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>esac</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>;;</div>
            <div>  *)</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>#echo "Usage: $SCRIPTNAME
                {start|stop|restart|reload|force-reload}" &gt;&amp;2
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>echo "Usage: $SCRIPTNAME
                {start|stop|restart|force-reload}" &gt;&amp;2
            </div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>exit 3</div>
            <div><span class="Apple-tab-span" style="white-space: pre;"> </span>;;</div>
            <div>esac</div>
            <div>:</div>
        </div>
        <div><br/></div>
        <div><b>Test the repcached</b></div>
        <div><br/></div>
        <div>In Server 1</div>
        <div>
            <ol style="text-align: left;">
                <li>telnet 127.0.0.1 11211</li>
                <li>set foo 0 0 3</li>
                <li>bar</li>
            </ol>
            <div>In Server 2</div>
        </div>
        <div>
            <ol style="text-align: left;">
                <li>telnet 127.0.0.1 11211</li>
                <li>get foo (You will get bar as the Output)</li>
            </ol>
        </div>
    </div>
</div>
<h2>Comments</h2>
<div class='comments'>
</div>
