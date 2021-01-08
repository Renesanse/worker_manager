#example
```
// Copyright Daniil Surnin. All rights reserved.
// Use of this source code is governed by a Apache license that can be
// found in the LICENSE file.

import 'dart:async';

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:http/http.dart';
import 'package:worker_manager/worker_manager.dart';

void main() async {
  /* this is not necessary, this code will spawn
   before your awesome widgets will build,
   to avoid micro freezes
   if you don't want to spawn free of calculation isolates,
   just don't write this code :
   ```WorkerManager().initManager()```*/
  await WorkerManager().initManager();
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      showPerformanceOverlay: true,
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage({Key key}) : super(key: key);
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  /*WorkerManager is Singleton. Just create link it everywhere you want*/
  WorkerManager workerManager = WorkerManager();
  int clicks = 0;
  List results = [];
  DateTime time;

  /*
    creating task for workerManager with global function and Bundle class for your function.
    bundle and timeout is optional parameters.
    */
  final task = Task(function: fib, bundle: 40, timeout: Duration(days: 78));

  /*remember, that you global function must have only one parameter, like int, String or your
    bundle class .
    For example:
    Class Bundle {
      final int age;
      final String name;
      Bundle(this.age, this.name);
    }
    optional parameters is ok, just be ready to avoid NPE
    */
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Container(
          height: 350,
          width: 200,
          color: Colors.cyan,
          child: Column(
            children: <Widget>[
              RaisedButton(
                child: Text('fib(40)'),
                onPressed: () async {
                  if (time == null) time = DateTime.now();
                  //manageWork function working with your task and returning stream which
                  //return result of your global function in listen callback
                  // also Stream from manage work handling errors
                  workerManager.manageWork(task: task).listen((sr) {
                    setState(() {
                      results.add(sr);
                    });
                    if (results.length == 3) {
                      print(DateTime.now().difference(time));
                    }
                  }).onError((error) {
                    print(error);
                  });
                  setState(() {
                    clicks++;
                  });
                },
              ),
              RaisedButton(
                child: Text('kill'),
                onPressed: () {
                  // killing task, stream will return nothing
                  workerManager.killTask(task: task);
                },
              ),
              Row(
                children: <Widget>[
                  Text(clicks.toString()),
                  CircularProgressIndicator(),
                  Text(results.length.toString())
                ],
              )
            ],
          ),
        ),
      ),
    );
  }

  @override
  void dispose() {
    // Good case when you want to end your hard calculations in dispose method
    workerManager.killTask(task: task);
    super.dispose();
  }
}

int fib(int n) {
  if (n < 2) {
    return n;
  }
  return fib(n - 2) + fib(n - 1);
}

Future<String> getData(String kek) async => (await get(kek)).body.toString();
```
