# AUTOSAR Explained Simply

> **Read this file first.** It teaches AUTOSAR using everyday comparisons — offices, post
> offices, notebooks, doctors. No prior knowledge needed.
>
> Once this makes sense, use `AUTOSAR_Complete_Guide.md` as your dictionary when you need
> exact details.

---

## Table of Contents

1. [The problem: a car is full of tiny computers](#1-the-problem)
2. [The one big idea (this is 80% of AUTOSAR)](#2-the-one-big-idea)
3. [The building with floors](#3-the-building-with-floors)
4. [Meet the cast: who does what](#4-meet-the-cast)
5. [The office: how software parts talk](#5-the-office)
6. [Software Components = employees](#6-software-components--employees)
7. [Two ways to talk: noticeboard vs phone call](#7-two-ways-to-talk)
8. [The timetable: when does my code run?](#8-the-timetable)
9. [The post office: how a message reaches another car computer](#9-the-post-office)
10. [Saving things: whiteboard vs notebook](#10-whiteboard-vs-notebook)
11. [When something breaks: the doctor and the medical file](#11-when-something-breaks)
12. [Going to sleep: the last one turns off the lights](#12-going-to-sleep)
13. [The supervisor who checks you're still working](#13-the-supervisor)
14. [Safety: the sealed envelope](#14-safety-the-sealed-envelope)
15. [How a real project is actually done](#15-how-a-real-project-is-done)
16. [Classic vs Adaptive: calculator vs smartphone](#16-classic-vs-adaptive)
17. [The 25 words you must know (plain English dictionary)](#17-the-25-words-you-must-know)
18. [One picture that holds everything](#18-one-picture)
19. [Test yourself](#19-test-yourself)
20. [What to do next](#20-what-to-do-next)

---

## 1. The problem

### A car is not one computer. It's about 100 of them.

Your car has a small computer for the engine. Another for the brakes. Another for the airbags.
One for each door. One for the wipers. One for the headlights. One for the radio.

Each little computer is called an **ECU**.

> **ECU** = Electronic Control Unit = one small computer inside the car.
> Just say "a small computer" in your head every time you see "ECU".

### What used to go wrong

Imagine you are a company that writes the wiper software. In the old days:

- You wrote the wiper software for **one specific chip**.
- The code that "thinks about wipers" was tangled together with the code that "pokes the chip".
- Next year the carmaker changes the chip → **you rewrite everything**.
- BMW wants it slightly different → **you write a second version**.
- Now do this for 100 computers, from 30 different suppliers, in one car.

The result was a mess. Very expensive. Very slow. Very hard to make safe.

```
   THE OLD WAY — everything glued together

   +--------------------------------------------+
   |  "If it's raining, move the wipers"        |   <- the idea (should be reusable)
   |  "Write value 0x3F to register TCCR1B"     |   <- chip-specific (not reusable)
   |  "Set pin 14 high"                         |   <- board-specific (not reusable)
   +--------------------------------------------+
              stuck to ONE chip forever
```

---

## 2. The one big idea

### Separate "what it does" from "which chip it runs on."

That's it. That's AUTOSAR. Everything else is detail.

### The phone charger comparison

Think about **USB-C**.

- Your phone doesn't know or care which brand of charger you use.
- The charger doesn't know or care which brand of phone you have.
- Why? Because there's a **standard plug** in the middle.

Anyone can build a phone. Anyone can build a charger. They just work together.

**AUTOSAR is the standard plug for car software.**

### The computer game comparison (even better)

You write a game for a PC.

- Do you write code for NVIDIA graphics cards? No.
- Do you write code for AMD graphics cards? No.
- You just say "draw this picture" and **Windows** handles the rest.
- The card maker supplies a **driver** that knows the hardware details.

So your game runs on any PC, unchanged.

Now map it:

| In a PC | In a car (AUTOSAR) |
|---|---|
| Your game | Your **application software** ("if raining, wipe") |
| Windows | The **middle layers** (RTE + Basic Software) |
| Graphics driver | The **chip drivers** (called MCAL) |
| The PC hardware | The car's small computer (ECU) |

**Remember this table.** If you understand it, you understand AUTOSAR's purpose.

### The official slogan

> *"Cooperate on standards, compete on implementation."*

In plain English: **"Let's all agree on the boring plumbing so we can compete on the interesting
stuff."** Nobody buys a car because its CAN driver has a nice internal structure.

### Two facts to remember

1. AUTOSAR started in **2003**, created by BMW, Bosch, Continental, Mercedes, VW and others.
2. AUTOSAR is **not a product you buy**. It's a rulebook. Companies like Vector and
   Elektrobit sell you software that *follows* the rulebook.

---

## 3. The building with floors

AUTOSAR software is organised in **floors**, like a building. Each floor has a job.

```
   +---------------------------------------------------+
   |  TOP FLOOR:  YOUR IDEAS                           |
   |  "If it's raining, turn on the wipers"            |
   |  Knows nothing about hardware.                    |
   +---------------------------------------------------+
   |  MIDDLE FLOOR:  THE MESSENGER                     |
   |  Carries messages between everyone.               |
   +---------------------------------------------------+
   |  LOWER FLOORS:  THE SERVICES                      |
   |  Storage, communication, diagnostics, timing.     |
   +---------------------------------------------------+
   |  GROUND FLOOR:  THE ELECTRICIANS                  |
   |  The only ones allowed to touch actual wires.     |
   +---------------------------------------------------+
   |  BASEMENT:  THE ACTUAL CHIP                       |
   +---------------------------------------------------+
```

### The one rule of the building

> **You may only talk to the floor directly below you.**

The top floor is not allowed to run down to the basement and start touching wires.
If it did, it would be stuck to that one basement forever — and we'd be back to the old mess.

### The real names (learn them slowly)

```
   +---------------------------------------------------+
   |  Application Layer          <- your ideas         |
   +---------------------------------------------------+
   |  RTE                        <- the messenger      |
   +---------------------------------------------------+
   |  Services Layer             \                     |
   +-------------------------------+  all of this is   |
   |  ECU Abstraction Layer        |  called "BSW"     |
   +-------------------------------+  (Basic Software) |
   |  MCAL                       /  <- electricians    |
   +---------------------------------------------------+
   |  Microcontroller            <- the chip           |
   +---------------------------------------------------+
```

| Name | Say it in your head as | Job |
|---|---|---|
| **Application Layer** | "my ideas" | The actual car feature |
| **RTE** | "the messenger" | Delivers everything between everyone |
| **BSW** (Basic Software) | "the plumbing" | Storage, network, diagnostics, timing |
| **MCAL** | "the electricians" | The only code that touches the chip |

**BSW** is just an umbrella word for the three lower floors together.

### Why "electricians" is a good word for MCAL

Electricians know which wire is which in **this particular building**. If you move to a new
building, you hire new electricians — but your **furniture stays the same**.

Same in AUTOSAR. Change the chip → swap the MCAL. Your application software doesn't change
at all. That's the win.

---

## 4. Meet the cast

There are about 80 standard modules in AUTOSAR. That sounds terrifying. It isn't — you only
need to know about 12 of them to hold a conversation.

Here they are as **people in a company**:

| Nickname | Real name | What they do |
|---|---|---|
| 🗣️ **The Messenger** | **RTE** | Carries every message between everyone |
| ⏰ **The Timetable** | **OS** | Decides who works, and when |
| 📮 **The Packer** | **Com** | Packs information into envelopes for sending |
| 🚦 **The Sorting Office** | **PduR** | Decides which envelope goes down which road |
| 🚌 **The Bus Driver** | **CanIf + Can** | Actually puts the message on the wire |
| 📓 **The Notebook Keeper** | **NvM** | Saves things that must survive a power-off |
| 🩺 **The Doctor** | **Dem** | Records faults, keeps the medical file |
| 💬 **The Receptionist** | **Dcm** | Talks to the mechanic's diagnostic tool |
| 🔌 **The Power Manager** | **EcuM** | Startup, shutdown, sleep, wake up |
| 🧠 **The Rule Book** | **BswM** | "If this happens, do that" |
| 📞 **The Phone Operator** | **ComM** | Decides when the network may be used |
| 👮 **The Supervisor** | **WdgM** | Checks that everyone is still doing their job |

That's the whole cast. Everything else is a supporting role.

---

## 5. The office

This is the most important chapter. Read it twice.

### Imagine a big office

You work in a big office. You need the sales figures from someone called Priya.

**The nice way to work:** you write "Priya, please send me the sales figures" and drop it in the
office message tray. It arrives. You don't know or care:

- Is Priya at the desk next to you?
- Is Priya in a different building?
- Did it go by hand, by email, or by courier?

**You just asked. It arrived.** Someone else solved the "how".

### That imaginary message tray is called the VFB

> **VFB** = Virtual Functional Bus
>
> Say it in your head as: **"the imaginary message tray where everyone can reach everyone."**

The word **virtual** is the key. The tray doesn't really exist. It's a **pretend** thing we use
while *designing*, so we can think about **who talks to whom** without worrying about **where
they physically sit**.

```
        THE VFB — the imaginary message tray (design time)

   [ Rain     ]   [ Speed    ]   [ Wiper    ]   [ Light    ]
   [ Sensor   ]   [ Sensor   ]   [ Control  ]   [ Control  ]
        |              |              |              |
   =====+==============+==============+==============+=====
              t h e   i m a g i n a r y   t r a y
   ========================================================

   Nobody knows where anybody sits. Nobody needs to.
```

### But someone has to actually deliver the message

The message tray is imaginary. In reality, somebody must physically walk the note over.

**That somebody is the RTE.**

> **RTE** = Runtime Environment
>
> Say it in your head as: **"the messenger who actually delivers things."**

### Here is the magic part

The messenger looks at the note and decides how to deliver it:

```
   Case 1: Priya sits at the next desk
   -----------------------------------
   The messenger just hands it over. Takes a moment. Done.
   (In real terms: it copies a variable in memory.)


   Case 2: Priya is in a different building
   -----------------------------------------
   The messenger puts it in an envelope, gives it to the post office,
   it travels by van, and someone at the other end delivers it to Priya.
   (In real terms: it goes out on the CAN network to another ECU.)
```

**Here's what matters: you wrote the exact same note in both cases.**

You didn't change a single word. You didn't even know which case it was.

This is called **location transparency**, and it's the whole reason AUTOSAR exists. It means:

- Your software can be **moved** to a different computer in the car without rewriting it.
- The carmaker can decide late in the project where things run.
- You can test your part before anyone has decided anything.

### Remember it like this

> **VFB = the plan. RTE = the reality.**
>
> The VFB is what you *draw on the whiteboard*.
> The RTE is the *code that gets generated* to make the drawing come true.

And one more crucial fact:

> **Nobody writes the RTE by hand.** A tool generates it automatically from your design.
> This is why AUTOSAR needs so many tools.

---

## 6. Software Components = employees

Your application software is split into pieces. Each piece is called a **Software Component**,
or **SWC** for short.

> **SWC** = Software Component = **one employee with one job.**

### What an employee looks like

```
        +--------------------------------------+
        |   EMPLOYEE:  "Wiper Controller"      |
        |                                      |
   IN --o  how hard is it raining?             |
        |                                      |
   IN --o  how fast is the car going?          |
        |                                      |
        |          how fast should the  o-- OUT|
        |          wipers move?                |
        +--------------------------------------+
```

The little circles on the sides are called **ports**. A port is just **a way in or a way out**.

- A port where information **comes in** = the employee **needs** something.
- A port where information **goes out** = the employee **provides** something.

### The two port names (and how to never forget them)

| Real name | Means | Memory trick |
|---|---|---|
| **P-Port** (Provide) | I **give** this out | **P** = **P**rovide = **P**roduce = sends out |
| **R-Port** (Require) | I **need** this | **R** = **R**equire = **R**eceive = takes in |

### The golden rule about employees

> An employee may **only** communicate through their ports. Never any other way.

No sneaking down to the basement. No secret side deals. Everything through the ports, and the
messenger (RTE) handles it. This is what makes the employee reusable — you can pick them up
and drop them into a completely different car.

### Groups of employees

You can put several employees in a **team**, and the team looks like one big employee from the
outside. A team is called a **Composition**.

```
   +===  TEAM: "Exterior Lights"  ===========+
   |                                          |
   |  [ Switch Reader ] --> [ Light Logic ]   |
   |                                          |
   +==========================================+
        From outside, this looks like one box.
```

Useful for organising a big project. But note: **the team is only a drawing**. When the code is
built, the team disappears and only the individual employees remain.

---

## 7. Two ways to talk

Employees talk to each other in two main ways. That's it — two.

### Way 1: The noticeboard 📋

You write today's temperature on the office noticeboard and walk away.

- You don't know who reads it.
- You don't wait for anyone.
- You get no reply.
- Anyone who's interested can look at it, any time.

**Real name: Sender-Receiver.**

Use it for: speed, temperature, "is the door open", "is it raining" — anything that is
**just a value**.

```
   [ Speed Sensor ] ---- "72 km/h" ----> [ Dashboard   ]
                                    \--> [ Cruise Ctrl ]
                                    \--> [ Wipers      ]
     one writer, as many readers as you like
```

#### Two flavours of noticeboard

| Flavour | Real name | What it's like |
|---|---|---|
| **Whiteboard** | *unqueued* / "data" | New value **rubs out** the old one. You only ever see the latest. Good for speed, temperature. |
| **Letterbox** | *queued* / "event" | Messages **pile up** in order. You read them one by one. Good for button presses — you don't want to miss one. |

```
   WHITEBOARD (unqueued)              LETTERBOX (queued)
   +--------+                         +----+----+----+
   |   75   |  <- 72 was rubbed out   | B1 | B2 | B3 |  <- all three kept
   +--------+                         +----+----+----+
   read it twice, same answer         each read takes one out
```

**Common exam question:** "What's the difference between `Rte_Write` and `Rte_Send`?"
Answer: **Write = whiteboard. Send = letterbox.** Same for `Rte_Read` (whiteboard) and
`Rte_Receive` (letterbox).

### Way 2: The phone call ☎️

You phone the accounts department and say "please calculate my tax". They do it and tell you
the answer.

- You know exactly who you called.
- You might wait for the answer.
- You **get a reply**.

**Real name: Client-Server.**

Use it for: "calculate this for me", "save this to memory", "report this fault" — anything
where you need something **done** and want to know **whether it worked**.

```
   [ Wiper SWC ] ---- "Please save this setting" ----> [ Memory Service ]
                <--- "Done, it worked" ---------------
```

> ⚠️ **A confusing detail worth knowing:** the one who **does** the work has the **P-Port**
> (they *provide* the service). The one who **asks** has the **R-Port**. So the port arrows
> and the "who calls whom" arrows point in opposite directions. This trips up everyone once.

### The other four ways (just so you've heard of them)

You'll meet these later. Don't memorise them now:

| Name | One-line meaning |
|---|---|
| **Mode Switch** | An announcement over the PA system: "The building is now in NIGHT mode." |
| **NvData** | A noticeboard whose contents survive a power cut (saved to memory). |
| **Parameter** | A settings sheet on the wall: "maximum wiper speed = 5". |
| **Trigger** | A bell that means "start now!" — no information, just a nudge. |

**Six ways in total.** But 90% of everything is the first two.

---

## 8. The timetable

### Your code does not run by itself

This surprises everyone. In AUTOSAR, **your software has no `main()` function**. It does not
start itself. It has no loop.

Instead you write small functions and say: *"please call this one every 10 milliseconds."*

Then something else calls them. Your code just sits there waiting to be called.

> **Runnable** = **one function that someone else calls for you.**
>
> Think of it as **one item on a to-do list** that a manager reads out.

### Who does the calling? The Operating System.

> **OS** = Operating System = **the school timetable.**

A school timetable says: *Maths at 9:00, Science at 10:00, Lunch at 12:00.* It doesn't care
what happens in the lessons. It just decides **who goes when**.

The AUTOSAR OS does exactly that for your functions.

### Tasks = periods on the timetable

A **task** is a slot in the timetable. Inside a slot, several of your functions run one after
another, always in the same order.

```
   THE 10-MILLISECOND SLOT   (this runs 100 times per second)
   +-----------------------------------------------+
   |  1st:  read the sensors                       |
   |  2nd:  work out the wiper speed               |
   |  3rd:  work out the light settings            |
   |  4th:  send everything out on the network     |
   +-----------------------------------------------+
```

A real car computer usually has slots like this:

```
   every 1 ms      -> fast things (engine, network handling)
   every 10 ms     -> most normal logic
   every 100 ms    -> slower things (diagnostics)
   every 1000 ms   -> very slow things (statistics, saving data)
   whenever free   -> background chores
```

### Priority: who interrupts whom

Every slot has a **priority number**. Higher number = more important.

If the 100 ms slot is running and the 1 ms slot becomes due, the 1 ms slot can **barge in**,
finish, and then the 100 ms one carries on. This is called **preemption** — think of it as
*"an urgent phone call interrupting your meeting."*

### What can trigger your function

Your function doesn't only run on a timer. It can also be triggered by:

| Trigger | Everyday meaning | Real name |
|---|---|---|
| A clock | "every 10 milliseconds" | **TimingEvent** ← most common by far |
| Data arriving | "run as soon as new speed arrives" | **DataReceivedEvent** |
| Someone phoning | "run because a client called me" | **OperationInvokedEvent** |
| Startup | "run once, at the very beginning" | **InitEvent** |
| Nothing to do | "run when the computer is idle" | **BackgroundEvent** |
| A mode change | "run when we switch to SLEEP" | **ModeSwitchEvent** |

### One rule that causes real bugs

> **Never wait around inside your function.**

If your function sits there waiting for something, the whole slot is blocked, the next slot is
late, the whole computer falls behind, and the supervisor (chapter 13) resets the car computer.

If something takes a long time, you **ask** for it now, and **check the answer next time round**.
Ask, don't wait.

---

## 9. The post office

Now let's follow one message all the way out of the car computer and into another one.

### The story

Your wiper software wants to tell the dashboard the car's speed. The dashboard is a
**different computer**, connected by a network cable.

Here's the journey, told as a post office:

```
 STEP 1   YOU WRITE THE NOTE
          "The speed is 72."
          You just call the messenger. You're done. Go do something else.
                  |
                  v
 STEP 2   THE MESSENGER (RTE) TAKES IT
          "This one has to go to another building. I'll send it by post."
                  |
                  v
 STEP 3   THE PACKER (Com) PUTS THINGS IN AN ENVELOPE
          An envelope holds 8 boxes. The packer decides:
          speed goes in boxes 1 and 2, engine RPM in boxes 3 and 4,
          some flags in box 5. Everything fits in ONE envelope.
                  |
                  v
 STEP 4   THE SORTING OFFICE (PduR) PICKS THE ROAD
          "This envelope goes on the CAN road, not the LIN road."
                  |
                  v
 STEP 5   THE LOADING BAY (CanIf) PICKS A VAN
          "Use loading bay number 3, it's free."
                  |
                  v
 STEP 6   THE DRIVER (Can driver) DRIVES
          Physically puts the electrical signals onto the two wires.
                  |
                  v
 ==================== THE WIRE ====================
                  |
                  v
 STEP 7   THE OTHER BUILDING RECEIVES IT
          The same chain, backwards: driver -> loading bay -> sorting office
          -> unpacker -> messenger -> the dashboard software reads "72".
```

### The real names for the chain

This is worth memorising. It's asked in almost every AUTOSAR interview:

```
   YOUR CODE  ->  RTE  ->  Com  ->  PduR  ->  CanIf  ->  Can  ->  the wire
   (you)      (messenger)(packer)(sorting) (loading  (driver)
                                  office)   bay)
```

And receiving is the same chain read backwards.

### Three words that confuse everyone

They're actually simple:

```
   SIGNAL  = one piece of information         "speed = 72"
             (this is what YOU care about)

   PDU     = the envelope, with several signals packed inside
             (this is what the middle of the system cares about)

   FRAME   = the envelope plus the address label, stamps and seals
             (this is what actually travels on the wire)
```

> **PDU** stands for "Protocol Data Unit". Ignore the words. **Just think "envelope".**

### Why bother packing things together?

Because sending an envelope costs time on the wire. If you sent every single number in its own
envelope, the network would be full of mostly-empty envelopes. So the packer bundles related
things together and sends fewer, fuller envelopes.

### What if the message is too big?

A CAN envelope only holds **8 boxes** (8 bytes). What if you need to send 200 boxes' worth,
like during a software update?

You **split it across many envelopes and number them**, exactly like sending a long document as
"page 1 of 30", "page 2 of 30". The receiver collects them all and staples them back together.

The module that does this is **CanTp** (Transport Protocol). *Say it as "the parcel splitter".*

The receiving end can also say "slow down, I can't keep up" — like a friend saying "wait, let me
catch up before you send more."

---

## 10. Whiteboard vs notebook

### The problem

Your car computer knows the odometer reading: 45,231 km.

You switch off the car. All the working memory is **wiped clean**. Tomorrow the odometer must
still say 45,231 km, not zero.

### The comparison

| | **Whiteboard** | **Notebook** |
|---|---|---|
| Real name | RAM | Flash / EEPROM |
| Speed | Instant | Slow |
| Survives power off? | ❌ No, wiped clean | ✅ Yes |
| How many times can you use it? | Unlimited | **Limited!** Wears out |

That last row is the surprising one. Flash memory physically **wears out** after being
rewritten too many times — think of a notebook page you can only erase and rewrite so often
before it tears.

### Who handles this

> **NvM** = the **notebook keeper**.
> (NvM stands for "Non-volatile Memory Manager". "Non-volatile" just means "doesn't vanish".)

You tell the notebook keeper: *"Please save this."* They handle everything else.

### Two things beginners get wrong

**Mistake 1: expecting it to happen instantly.**

When you say "save this", the notebook keeper says **"noted, I'll do it shortly"** and walks
away. The writing happens later, in the background. If you sit there waiting for it, you'll
block your whole timetable slot (see chapter 8's rule).

So: **ask to save, then check later whether it's done.**

**Mistake 2: not understanding the wear problem.**

Because flash wears out, there's a clever module underneath that **spreads the writing around**
instead of hammering the same spot. It's like a notebook where, instead of erasing page 1 over
and over, you just write on a fresh page each time and cross out the old one — and only
occasionally copy everything into a clean notebook.

That module is called **Fee** ("Flash EEPROM Emulation"). *Say it as "the page manager".*

### The chain

```
   YOUR CODE  ->  NvM         ->  MemIf      ->  Fee            ->  Fls   -> the flash chip
   (you)          (notebook       (which          (page             (the
                   keeper)         notebook?)      manager)          pen)
```

### When it happens

```
   Car starts  ->  read everything from the notebook onto the whiteboard
   Car running ->  work on the whiteboard (fast)
   Car stops   ->  copy the changed bits back into the notebook
```

That last step is why a car computer stays powered for a moment after you switch off — it's
finishing its writing.

---

## 11. When something breaks

### The story

A temperature sensor fails. Three things need to happen:

1. **Remember it** — write it in the car's medical file.
2. **React to it** — stop using that sensor, use a fallback.
3. **Tell someone** — when a mechanic plugs in their tool, show them the problem.

### The doctor and the medical file

> **Dem** = the **doctor** who keeps the **medical file**.
> (Dem = "Diagnostic Event Manager".)

When your software notices something wrong, it tells the doctor:
*"the coolant sensor is reading nonsense."*

The doctor is sensible about it:

**It doesn't panic at one symptom.** If the sensor glitches once, that could be electrical
noise. The doctor waits until it's happened enough times to be convinced. (Real name:
**debouncing**. Just think *"don't panic at one glitch."*)

**It writes a fault code.** A short code like `P0128`. This is called a **DTC** —
Diagnostic Trouble Code. *Just think "fault code".*

**It takes a snapshot.** It records what the car was doing at that exact moment: speed,
engine RPM, temperature, how long the engine had been running. This is enormously useful for a
mechanic later. (Real name: **freeze frame**. Think *"a photograph of the moment".*)

**It saves it to the notebook** so it survives power-off.

**It can forgive.** If the fault doesn't come back for a while, the doctor eventually erases it.
(Real name: **aging**. Think *"the fault heals".*)

### The receptionist

> **Dcm** = the **receptionist** who talks to the mechanic's tool.
> (Dcm = "Diagnostic Communication Manager".)

When a mechanic plugs their diagnostic tool into your car, they're having a conversation. The
receptionist handles it.

The conversation has a fixed set of phrases, standardised worldwide. It's called **UDS**.

The most common phrases:

| The mechanic asks | Real name | Code |
|---|---|---|
| "What faults do you have?" | ReadDTCInformation | 0x19 |
| "Tell me your part number / VIN / this sensor's value" | ReadDataByIdentifier | 0x22 |
| "Clear all the faults" | ClearDiagnosticInformation | 0x14 |
| "Turn the cooling fan on right now, so I can test it" | InputOutputControl | 0x2F |
| "Run your self-test" | RoutineControl | 0x31 |
| "Let me in, here's the password" | SecurityAccess | 0x27 |
| "I'm about to install new software" | RequestDownload | 0x34 |
| "I'm still here, don't hang up" | TesterPresent | 0x3E |

The car answers either **"here's your answer"** or **"no, because…"** with a reason code.

Two reason codes worth knowing:

- **0x31** = "I don't know what you're asking about."
- **0x78** = **"Hang on, I'm working on it, don't give up on me."** ← this one is very common.
  The car uses it when an answer takes longer than usual.

### The third piece

> **FiM** = the manager who **switches off features that can't work any more**.

If the wheel speed sensor is broken, cruise control must be disabled. FiM keeps the list of
"if this is broken, turn that off."

---

## 12. Going to sleep

### The problem

Your car is parked overnight. If every one of the 100 computers stayed fully awake, the battery
would be flat by morning. So they must all sleep.

But here's the catch: **they share the same network cable.** If one computer is still talking,
the others can't sleep — they'd miss the message.

### The meeting room comparison

Picture a meeting room with a light switch.

- People come and go.
- The **last person to leave** turns off the light.
- If someone's still in there working, the light stays on.
- If anyone walks back in, the light comes back on.

That's exactly it.

### How they agree

Every computer that still needs the network sends a tiny "**I'm still here**" message,
regularly. Like people occasionally saying something so you know they're in the room.

```
   Someone needs the network   ->  they keep saying "still here"
   They finish                 ->  they stop saying it, but keep listening
   Nobody has said anything    ->  short pause, then...
   for a while                     ...everybody sleeps. Lights out. 🌙

   Anyone needs it again       ->  they say "still here" -> everybody wakes up
```

**Real name: Network Management (NM).**

### A clever refinement

Sometimes you only need *part* of the car awake. Pressing the key fob to unlock needs the door
computers — it doesn't need the engine computer.

So the messages carry a little list saying *which group* is needed. Only those computers wake
up. The rest stay asleep and save battery.

**Real name: Partial Networking.** *Think: "wake only the people you actually need."*

### Who else is involved

| Nickname | Real name | Job |
|---|---|---|
| The power manager | **EcuM** | Handles the actual startup, shutdown and wake-up of this computer |
| The phone operator | **ComM** | Decides whether this computer is allowed to use the network right now |
| The rule book | **BswM** | "If A and B are true, then do C." The general decision-maker |

**BswM** deserves a note. It's basically a list of **if-then rules** that you *configure*
rather than *program*:

```
   IF the network is active AND we are in normal driving mode
   THEN start sending the engine messages

   IF the mechanic has started a software update
   THEN stop all normal messages
```

---

## 13. The supervisor

### The problem

What if your software **freezes**? Stuck in a loop, crashed, waiting for something that never
comes. A frozen brake controller is genuinely dangerous.

### The solution: a supervisor who won't be fooled

There is a piece of **hardware** — completely separate from your software — that must be
"petted" regularly. If it doesn't get petted, it assumes everything has gone wrong and
**restarts the whole computer**.

It's called a **watchdog**. Think of it as **a dog that must be fed every few seconds. Miss a
feeding, and it bites (resets the computer).**

### But there's a smarter version

Just feeding the dog isn't enough. What if your software is running, but doing the *wrong*
thing? So AUTOSAR has a supervisor that checks three things:

| What it checks | Everyday version |
|---|---|
| **Are you running often enough?** | "You're supposed to report every 10 minutes. It's been an hour." |
| **Did you finish in time?** | "You started this task at 9:00 and it's now 5pm. Something's wrong." |
| **Did you do the steps in the right order?** | "You submitted the form before filling it in. That's not possible." |

If any check fails, the supervisor **stops feeding the dog**, and the dog resets the computer.

**Real name: WdgM** (Watchdog Manager). Your code reports "I got here" at various points —
these are called **checkpoints** — and the supervisor watches the pattern.

---

## 14. Safety: the sealed envelope

### The problem

You send "brake pressure = 80%" to another computer. What if it arrives as "8%"?

Wires get noisy. Software has bugs. Messages get lost, arrive twice, or arrive out of order.
For something like braking, a corrupted message could genuinely hurt someone.

### The solution: three simple additions

Think of sending an important legal document by post. You would:

| What you'd do | What it protects against | Real name |
|---|---|---|
| Add a **tamper-proof seal** | Someone changed the contents | **CRC** (a checksum) |
| Add **"page 5 of 12"** | A page is missing, duplicated, or out of order | **Counter** |
| Write **"this is the brake document"** on it | You got the wrong document entirely | **Data ID** |

Put all three on the message, and the receiver can check: *seal intact? page number sensible?
right document?* If any check fails, the receiver knows not to trust it.

**Real name: E2E protection** (End-to-End protection).

### Why "end-to-end" matters

The message passes through many hands: packer, sorting office, driver, wire, and back up again.
Any of them *could* have a bug.

The seal is applied by the **original sender** and checked by the **final receiver**. So it
doesn't matter what happened in between — if anything went wrong anywhere along the way, it
gets caught.

That's why it's called end-to-end: **checked from the very start to the very end**, not at each
step.

### A word you'll hear: ASIL

Safety requirements come in levels, from **QM** (no safety requirement — the radio volume) up to
**ASIL D** (the most serious — brakes, airbags, steering).

The higher the level, the more evidence you must produce that your software is correct. This
comes from a standard called **ISO 26262**. That's all you need to know for now.

### One more idea: keeping things apart

On one computer you might run both an ASIL D brake function **and** a QM ambient-lighting
function. You must be certain the lighting code can never corrupt the brake code.

So the chip has hardware walls between them — the lighting software physically **cannot** write
into the brake software's memory. Think of it as **separate locked rooms in the same building**.

The industry phrase is **"freedom from interference"** — which just means *"the unimportant
stuff can't break the important stuff."*

---

## 15. How a real project is done

### The LEGO instruction comparison

Building an AUTOSAR project is like building a huge LEGO set where different people build
different sections.

```
 STEP 1  THE ARCHITECT DRAWS THE PLAN
         "We need these 40 employees. This one talks to that one."
         Nobody has decided which computer they run on yet.
                     |
                     v
 STEP 2  SOMEONE DECIDES WHO SITS WHERE
         "Wiper Control goes on the Body Computer."
         "The speed value travels on network message number 0x123."
                     |
                     v
 STEP 3  CUT OUT ONE COMPUTER'S SHARE
         Take the big plan and extract only the parts for the Body Computer.
         Hand that file to the supplier who builds the Body Computer.
                     |
                     v
 STEP 4  FILL IN THOUSANDS OF SETTINGS      <-- MOST OF THE REAL WORK
         "This slot runs every 10 ms."
         "This value sits in byte 2 of that envelope."
         "This fault gets code P0128."
         "Pin 14 is a PWM output."
                     |
                     v
 STEP 5  PRESS "GENERATE"
         The tool reads all those settings and writes thousands of
         lines of C code automatically — including the messenger (RTE).
                     |
                     v
 STEP 6  ADD YOUR OWN CODE AND BUILD IT
         Generated code + bought-in software + your logic
         -> compile -> one file -> load it onto the computer.
```

### The surprising truth about the job

> Most AUTOSAR work is **configuring**, not **programming**.

A large part of the day is spent in a tool with thousands of settings, deciding how things
should be wired together — and then pressing "generate".

The actual C code you write by hand is often a small fraction of the total.

### The file format

All these plans and settings are stored in files called **ARXML** — just XML files with a
standard structure. Tools from different companies can all read them. That's the point.

You rarely edit them by hand. Tools do it for you.

### Who does what in a real team

| Role | What they do all day |
|---|---|
| **System architect** (at the carmaker) | Draws the big plan, decides which computer does what |
| **Function developer** | Writes the actual feature logic (often in Simulink) |
| **BSW integrator** | Fills in all those settings, generates, builds, flashes ← very common job |
| **Diagnostics engineer** | Fault codes, the mechanic-tool conversation |
| **Test engineer** | Proves it all works |

---

## 16. Classic vs Adaptive

There are two kinds of AUTOSAR. Beginners find this confusing, so here it is simply.

### The comparison

> **Classic AUTOSAR is a pocket calculator.**
> **Adaptive AUTOSAR is a smartphone.**

A calculator:
- Does a few things, perfectly, instantly, forever.
- You can't install apps on it.
- It never crashes.
- It costs almost nothing and uses almost no power.

A smartphone:
- Does enormously complicated things.
- You install and remove apps whenever you like.
- It updates itself overnight.
- It needs a lot of memory and power.

**A car needs both.** You want the brakes to be a calculator. You want the self-driving system
to be a smartphone.

### Side by side

| | **Classic** (the calculator) | **Adaptive** (the smartphone) |
|---|---|---|
| Used for | Brakes, engine, airbag, doors, lights | Self-driving, cameras, infotainment, connectivity |
| Since | 2003 | 2017 |
| Language | C | C++ |
| Memory | Tiny (kilobytes) | Large (gigabytes) |
| Everything decided | **Before** the software is built | Partly **while it's running** |
| Add software later? | No — reflash the whole thing | Yes — install apps, update over the air |
| How parts find each other | Fixed at design time | They **look for each other** while running |
| Speed of reaction | Microseconds | Milliseconds |

### The one genuinely different idea in Adaptive

In Classic, everything is decided in advance: *"the speed value is in byte 2 of message 0x123,
forever."*

In Adaptive, software parts **announce themselves** and **look for each other** while the car
is running:

```
   One app says:      "Hello, I offer a Radar service."
   Another app says:  "I'm looking for a Radar service."
   The system:        introduces them. They start talking.
```

This is called **service-oriented** communication, and it's basically how apps on your phone
and services on the internet work.

### Which will you work with?

Most AUTOSAR jobs today are **Classic**. Adaptive is growing quickly and is where the
self-driving and connected-car work happens. Learn Classic first — Adaptive makes far more sense
once you understand what it's reacting against.

---

## 17. The 25 words you must know

Cover the right column and test yourself.

| Word | Plain English meaning |
|---|---|
| **AUTOSAR** | A rulebook so car software can be reused across different chips and carmakers |
| **ECU** | One small computer in the car |
| **SWC** | One employee with one job (a piece of application software) |
| **Port** | A way in or out of an employee |
| **VFB** | The imaginary message tray used while designing |
| **RTE** | The messenger who actually delivers things |
| **BSW** | The plumbing — all the standard supporting software |
| **MCAL** | The electricians — the only code that touches the chip |
| **CDD** | A special permission slip to break the rules when you really must |
| **Runnable** | One function that the system calls for you |
| **Task** | A slot in the timetable where several functions run |
| **OS** | The timetable — decides who runs when |
| **Com** | The packer — puts information into envelopes |
| **PDU** | An envelope |
| **Signal** | One piece of information inside the envelope |
| **PduR** | The sorting office — picks which road the envelope takes |
| **CAN** | The most common network cable in a car |
| **CanTp** | The parcel splitter — for messages too big for one envelope |
| **NvM** | The notebook keeper — saves things that must survive power-off |
| **Dem** | The doctor — records faults in a medical file |
| **DTC** | A fault code, like `P0128` |
| **Dcm** | The receptionist — talks to the mechanic's diagnostic tool |
| **UDS** | The standard set of phrases the mechanic's tool uses |
| **WdgM** | The supervisor — checks everyone is still working properly |
| **E2E** | The tamper-proof seal + page numbers on important messages |

### Bonus five

| Word | Plain English meaning |
|---|---|
| **EcuM** | The power manager — startup, shutdown, sleep, wake |
| **BswM** | The rule book — "if this, then that" |
| **ComM** | The phone operator — may we use the network right now? |
| **ARXML** | The file format that all the tools read and write |
| **ASIL** | How safety-critical something is (QM is lowest, ASIL D is highest) |

---

## 18. One picture

If you remember nothing else, remember this.

```
   +==================================================================+
   |   YOUR IDEAS                                                     |
   |   "If it's raining, move the wipers"                             |
   |   Employees (SWCs) talking through ports.                        |
   |   Knows NOTHING about hardware.                                  |
   +==================================================================+
   |   THE MESSENGER (RTE)                                            |
   |   Delivers every message. Hides where everyone sits.             |
   |   Generated automatically. Nobody writes it by hand.             |
   +==================================================================+
   |   THE PLUMBING (BSW)                                             |
   |                                                                  |
   |   Timetable (OS)   Packer (Com)    Notebook (NvM)                |
   |   Doctor (Dem)     Receptionist (Dcm)   Supervisor (WdgM)        |
   |   Power (EcuM)     Rules (BswM)    Sorting office (PduR)         |
   +==================================================================+
   |   THE ELECTRICIANS (MCAL)                                        |
   |   The only ones allowed to touch the chip.                       |
   |   Supplied by the chip maker. Swap these when the chip changes.  |
   +==================================================================+
   |   THE CHIP                                                       |
   +==================================================================+

   RULE: you may only talk to the box directly below you.

   THE POINT: change the chip, swap only the bottom box.
              Everything above it stays exactly the same.
```

And the one journey to remember:

```
   YOU  ->  MESSENGER  ->  PACKER  ->  SORTING OFFICE  ->  DRIVER  ->  the wire
   ---      ---------      ------      --------------      ------
   SWC        RTE           Com            PduR         CanIf + Can
```

---

## 19. Test yourself

Try answering out loud before looking. If you can do these, you genuinely understand AUTOSAR.

**1. In one sentence, why does AUTOSAR exist?**
So that car software can be moved between different chips, computers and carmakers without
rewriting it.

**2. What is the VFB?**
The imaginary message tray used while designing, where every part can reach every other part
without knowing where anything physically sits.

**3. What is the RTE?**
The messenger. It's generated code that actually delivers the messages the VFB only pretended
to deliver.

**4. Why can't your software talk to the chip directly?**
Because then it would be stuck to that one chip — which is the exact problem AUTOSAR was
invented to solve.

**5. Noticeboard or phone call — which is which?**
Noticeboard = Sender-Receiver (just a value, no reply). Phone call = Client-Server (ask for
something, get an answer).

**6. Whiteboard or letterbox?**
Whiteboard = unqueued, the new value replaces the old. Letterbox = queued, messages pile up in
order.

**7. What is a runnable?**
A function you write that the system calls for you — usually on a timer. Your code never starts
itself.

**8. Trace a message from your code to the wire.**
You → RTE → Com → PduR → CanIf → Can → wire.

**9. What's the difference between a signal, a PDU and a frame?**
Signal = one piece of information. PDU = the envelope holding several signals. Frame = the
envelope plus addressing, actually travelling on the wire.

**10. Why can't you just write straight to flash memory?**
It's slow and it physically wears out. The notebook keeper (NvM) and page manager (Fee) handle
it properly, spreading writes around.

**11. What does the doctor (Dem) do?**
Records faults as codes, waits before believing a single glitch, takes a snapshot of conditions,
and saves it all so a mechanic can read it later.

**12. Why does the car network need to agree about sleeping?**
Because they share one cable. If one computer is still talking, nobody else can sleep — so the
last one out turns off the lights.

**13. What does the watchdog do?**
It must be fed regularly. If the software freezes and stops feeding it, it resets the computer.

**14. What are the three parts of E2E protection?**
A seal (CRC), a page number (counter), and a document name (Data ID). They catch corruption,
loss/duplication/reordering, and wrong-message delivery.

**15. Classic or Adaptive — which is which?**
Classic = the calculator: small, instant, fixed, for brakes and engines. Adaptive = the
smartphone: large, flexible, updatable, for self-driving and infotainment.

---

## 20. What to do next

### This week
Read this file once more. Then try to **draw the building** (chapter 18) on a blank sheet of
paper without looking. Do it three times over three days. When it comes out right every time,
you've got the foundation.

### Next week
Explain chapter 5 (the office, VFB and RTE) out loud to someone who knows nothing about cars.
If you can make them understand it, you understand it.

### After that
Open `AUTOSAR_Complete_Guide.md` and read **just Part A and Part B**. It will feel much easier
now, because you'll recognise the ideas — you'll just be learning the proper names for things
you already understand.

### Then
Pick **one** chapter of the big guide per week. Suggested order:

1. The Operating System (the timetable, in detail)
2. The communication stack (the post office, in detail)
3. Memory (the notebook, in detail)
4. Diagnostics (the doctor, in detail)

### Eventually
Get your hands on the real thing. A cheap board like an **NXP S32K144** plus an evaluation
version of a configuration tool will teach you more in a weekend than a month of reading.

Build this: **press a button → your code notices → an LED lights up → send the state on the
network → watch it appear in a network monitoring tool.** That tiny project touches almost
every idea in this document.

---

## One last thought

AUTOSAR feels overwhelming because it's mostly **vocabulary**. There are hundreds of
three-letter names, and they're all unfamiliar at once.

But the actual ideas underneath are things you already understand: a messenger, a timetable, a
post office, a notebook, a doctor, a supervisor.

Learn the ideas first. The names will stick to them on their own.
