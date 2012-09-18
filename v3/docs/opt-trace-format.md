# Execution Trace Format

This document describes the execution trace format that serves as the
interface between the frontend and backend of Online Python Tutor
(thereafter abbreviated as OPT).

It is a starting point for anyone who wants to create a different
backend (e.g., for another programming language) or a different frontend
(e.g., for visually-impaired students). View it online at:

https://github.com/pgbovine/OnlinePythonTutor/blob/master/v3/docs/opt-trace-format.md

Look at the Git history to see when this document was last updated; the
more time elapsed since that date, the more likely things are
out-of-date.

I'm assuming that you're competent in Python, JSON, command-line-fu, and
Google-fu. Feel free to email philip@pgbovine.net if you have questions.

And please excuse the sloppy writing; I'm not trying to win any style awards here :)


## Trace Overview

Before you continue reading, I suggest for you to first skim the Overview for Developers doc:
https://github.com/pgbovine/OnlinePythonTutor/blob/master/v3/docs/developer-overview.md

Pay particular attention to what `generate_json_trace.py` is and how to run it:
https://github.com/pgbovine/OnlinePythonTutor/blob/master/v3/docs/developer-overview.md#two-quick-tips-for-starters

Let's start with a simple example. Create an `example.py` file with the following contents:
```python
x = 5
y = 10
z = x + y
```

Now run:
```
python generate_json_trace.py example.py
```

and you should see the following output:
```javascript
{
  "code": "x = 5\ny = 10\nz = x + y\n\n", 
  "trace": [
    {
      "ordered_globals": [], 
      "stdout": "", 
      "func_name": "<module>", 
      "stack_to_render": [], 
      "globals": {}, 
      "heap": {}, 
      "line": 1, 
      "event": "step_line"
    }, 
    {
      "ordered_globals": [
        "x"
      ], 
      "stdout": "", 
      "func_name": "<module>", 
      "stack_to_render": [], 
      "globals": {
        "x": 5
      }, 
      "heap": {}, 
      "line": 2, 
      "event": "step_line"
    }, 
    {
      "ordered_globals": [
        "x", 
        "y"
      ], 
      "stdout": "", 
      "func_name": "<module>", 
      "stack_to_render": [], 
      "globals": {
        "y": 10, 
        "x": 5
      }, 
      "heap": {}, 
      "line": 3, 
      "event": "step_line"
    }, 
    {
      "ordered_globals": [
        "x", 
        "y", 
        "z"
      ], 
      "stdout": "", 
      "func_name": "<module>", 
      "stack_to_render": [], 
      "globals": {
        "y": 10, 
        "x": 5, 
        "z": 15
      }, 
      "heap": {}, 
      "line": 3, 
      "event": "return"
    }
  ]
}
```

Recall that when OPT is deployed on a webserver, the backend generates this trace and sends it to the frontend,
where it will be turned into a visualization.

[Click here](http://pythontutor.com/visualize.html#code=x+%3D+5%0Ay+%3D+10%0Az+%3D+x+%2B+y&mode=display&cumulative=false&py=2&curInstr=0)
to see the visualization of this trace (open it in a new window if possible).
Note that the trace object contains *all* of the information required to create this visualization.

The trace is a JSON object with two fields: `code` is the string contents of the code
to be visualized, and `trace` is the actual execution trace, which consists of a list of execution points.

In the above example, `trace` is a list of four elements since there are four execution points.
If you step through the visualization, you'll notice that there are exactly four steps, one for each
element of the `trace` list.
(Sometimes the frontend will filter out some redundant entries in `trace`, but a simplifying assumption
is that `trace.length` is the number of execution steps that the frontend renders.)

Ok, still with me? Let's now dig into what an individual element in `trace` looks like.


## Execution Point Objects

The central type of object in a trace is an "execution point", which represents the state of the computer's (abstract)
memory at a certain point in execution. Recall that a trace is an ordered list of execution points.

The key concept to understand is that the frontend renders an execution point by simply looking at
the contents of the corresponding execution point object, **without consulting any of its neighbors**.

Ok, let's now look at the **four** execution points in our above example in order. The first point
is what the frontend visualizes when it says "Step 1 of 3":

```javascript
    {
      "ordered_globals": [], 
      "stdout": "", 
      "func_name": "<module>", 
      "stack_to_render": [], 
      "globals": {}, 
      "heap": {}, 
      "line": 1, 
      "event": "step_line"
    }
```

This is pretty much what an "empty" execution point object looks like. `line` shows the line number of the
line that is *about to execute*, which is line 1 in this case. And `event` is `step_line`, which indicates
that an ordinary single-line step event is about to occur. `func_name` is the function that's currently
executing: In this case, `<module>` is the faux name for top-level code that's not in any function.
All of the other fields are empty, and if you look at the visualization, nothing is rendered in the "Frames"
or "Objects" panes.

Ok now let's look at the second point, which corresponds to the frontend visualization when it says
"Step 2 of 3":

```javascript
    {
      "ordered_globals": [
        "x"
      ], 
      "stdout": "", 
      "func_name": "<module>", 
      "stack_to_render": [], 
      "globals": {
        "x": 5
      }, 
      "heap": {}, 
      "line": 2, 
      "event": "step_line"
    }
```



```javascript
    {
      "ordered_globals": [
        "x", 
        "y"
      ], 
      "stdout": "", 
      "func_name": "<module>", 
      "stack_to_render": [], 
      "globals": {
        "y": 10, 
        "x": 5
      }, 
      "heap": {}, 
      "line": 3, 
      "event": "step_line"
    }
```

```javascript
    {
      "ordered_globals": [
        "x", 
        "y", 
        "z"
      ], 
      "stdout": "", 
      "func_name": "<module>", 
      "stack_to_render": [], 
      "globals": {
        "y": 10, 
        "x": 5, 
        "z": 15
      }, 
      "heap": {}, 
      "line": 3, 
      "event": "return"
    }
```