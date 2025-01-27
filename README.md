# LLM Prompt Templates for Manim

[Manim](https://github.com/3b1b/manim) is an animation engine for rendering math videos, created by and for 3Blue1Brown.

But, there is a [community edition](https://github.com/ManimCommunity/manim/), built to be more stable and well tested.

## Why Prompt Templates?

Most LLMs in 2025 have a knowledge cutoff, and get confused with API changes. You need to usually state changes.

This is an attempt to collect some of those changes into a single, copy pastable file.

For example, here's the result for the following prompt on Claude 3.5:

```
can you create a manim animation with a ball that bounces from the top to the bottom? made it red
```

It writes code, but doesn't run:

```
test_prompt.py:29 in        │
│ construct                                                                    │
│                                                                              │
│   26 │   │   │   │   ApplyMethod(                                            │
│   27 │   │   │   │   │   ball.move_to,                                       │
│   28 │   │   │   │   │   end_pos,                                            │
│ ❱ 29 │   │   │   │   │   rate_func=rate_functions.ease_in,                   │
│   30 │   │   │   │   │   run_time=time_per_bounce/2                          │
│   31 │   │   │   │   )                                                       │
│   32 │   │   │   )                                                           │
╰──────────────────────────────────────────────────────────────────────────────╯
AttributeError: module 'manim.utils.rate_functions' has no attribute 'ease_in'
[57409] Execution returned code=1 in 0.722 seconds returned signal null 
```

Now, if we add some context about the changes:

```
that's an older api. this should be more recent:

.. manim:: RateFuncExample
    :save_last_frame:
    class RateFuncExample(Scene):
        def construct(self):
            x = VGroup()
            for k, v in rate_functions.dict.items():
                if "function" in str(v):
                    if (
                        not k.startswith("")
                        and not k.startswith("sqrt")
                        and not k.startswith("bezier")
                    ):
                        try:
                            rate_func = v
                            plot = (
                                ParametricFunction(
                                    lambda x: [x, rate_func(x), 0],
                                    t_range=[0, 1, .01],
                                    use_smoothing=False,
                                    color=YELLOW,
                                )
                                .stretch_to_fit_width(1.5)
                                .stretch_to_fit_height(1)
                            )
                            plot_bg = SurroundingRectangle(plot).set_color(WHITE)
                            plot_title = (
                                Text(rate_func.name__, weight=BOLD)
                                .scale(0.5)
                                .next_to(plot_bg, UP, buff=0.1)
                            )
                            x.add(VGroup(plot_bg, plot, plot_title))
                        except: # because functions not_quite_there, function squish_rate_func are not working.
                            pass
            x.arrange_in_grid(cols=8)
            x.height = config.frame_height
            x.width = config.frame_width
            x.move_to(ORIGIN).scale(0.95)
            self.add(x)
There are primarily 3 kinds of standard easing functions:
#. Ease In - The animation has a smooth start.
#. Ease Out - The animation has a smooth end.
#. Ease In Out - The animation has a smooth start as well as smooth end.
.. note:: The standard functions are not exported, so to use them you do something like this:
    rate_func=rate_functions.ease_in_sine
    On the other hand, the non-standard functions, which are used more commonly, are exported and can be used directly.
.. manim:: RateFunctions1Example
    class RateFunctions1Example(Scene):
        def construct(self):
            line1 = Line(3LEFT, 3RIGHT).shift(UP).set_color(RED)
            line2 = Line(3LEFT, 3RIGHT).set_color(GREEN)
            line3 = Line(3LEFT, 3RIGHT).shift(DOWN).set_color(BLUE)
            dot1 = Dot().move_to(line1.get_left())
            dot2 = Dot().move_to(line2.get_left())
            dot3 = Dot().move_to(line3.get_left())
            label1 = Tex("Ease In").next_to(line1, RIGHT)
            label2 = Tex("Ease out").next_to(line2, RIGHT)
            label3 = Tex("Ease In Out").next_to(line3, RIGHT)
            self.play(
                FadeIn(VGroup(line1, line2, line3)),
                FadeIn(VGroup(dot1, dot2, dot3)),
                Write(VGroup(label1, label2, label3)),
            )
            self.play(
                MoveAlongPath(dot1, line1, rate_func=rate_functions.ease_in_sine),
                MoveAlongPath(dot2, line2, rate_func=rate_functions.ease_out_sine),
                MoveAlongPath(dot3, line3, rate_func=rate_functions.ease_in_out_sine),
                run_time=7
            )
            self.wait()
"""
```

We get a working program. This repo is about automating that process into a single prompt you can paste into your chat.
