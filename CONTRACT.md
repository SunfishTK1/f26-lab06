# Contract Worksheet

One section per milestone. Fill each one in as you go, in order. Write each
prediction before you run anything. That is the part a TA asks about.

Keep it short and specific. Point at methods, call sites, and error text.

---

## Milestone 1: The notes overload

### Prediction (write this before you run the build, and you can deliberate with your agent)

**Will the consumer, untouched, still compile and pass?** Yes or no.

MY ANSWER: YES

**Why.** What does the compiler do with the consumer's existing call sites once
the new overload exists?

MY ANSWER WHY: Both of the consumer's createBooking calls pass 4 arguements, so Java can still use the 4-arguement method, and not need to consider the 5-arguement overload.  The only class that has to add new methods is InMemoryBooking Service in api/, since FrontDesk just calls the interface and doesn not implemnt it, so nothing in consumer breaks.

### What happened

**The result.** What the build printed for each module.

Change: added `createBooking(String roomId, long startMinute, long endMinute,
String waitlistKey, String notes)` to `BookingApi`, and `getNotes()` to
`Booking`. The old 4-arg `createBooking` in `InMemoryBookingService` now
delegates with `notes = null`. Nothing under `consumer/` changed.

`mvn -B clean test`:

```
[INFO] Building lab06-api 1.0.0                                           [2/3]
[INFO] Compiling 4 source files with javac [debug deprecation release 21] to target/classes
[INFO] Compiling 1 source file with javac [debug deprecation release 21] to target/test-classes
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO] Building lab06-consumer 1.0.0                                      [3/3]
[INFO] Compiling 1 source file with javac [debug deprecation release 21] to target/classes
[INFO] Compiling 1 source file with javac [debug deprecation release 21] to target/test-classes
[INFO] Tests run: 7, Failures: 0, Errors: 0, Skipped: 0
[INFO] lab06-booking-parent ............................... SUCCESS
[INFO] lab06-api .......................................... SUCCESS
[INFO] lab06-consumer ..................................... SUCCESS
[INFO] BUILD SUCCESS
```

**If your prediction was wrong,** say what you missed.

It was right.

**Is an additive change always safe in Java?** One case where adding something
to an API still breaks a caller, if you can name one.

MY ANSWER: A new overload that has the same number of parameters with a null argument can make it ambigious what is being called.

---

## Milestone 2: The request object

### Prediction (write this before you run the build)

**Will the untouched consumer still compile and pass?** Yes or no, and if no,
which module goes red and whether at compile time or test time.

MY Answer: No. because the consumer will likelly fail.  since createBooking is removed.

**Where.** Name the call sites you expect to be affected, if any.

api.createBooking is affected

**What about the tests in `api/`, after you update them?** And whether their
result is evidence about the consumer.

Those should pass because they call a method that keepsmthe same behavior.


### Step 1: after the fold

**What the build printed.** Paste it for each module, including file and
line for anything that failed.

Change: removed both positional `createBooking` methods from `BookingApi` and
`InMemoryBookingService`, and added `createBooking(BookingRequest)` with a new
immutable `BookingRequest` (`BookingRequest.of(roomId, start, end)` plus
`withWaitlistKey(...)` / `withNotes(...)`). Rewrote all five api tests to the
new call. Nothing under `consumer/` changed.

`mvn -B clean test` (absolute path prefix trimmed):

```
[INFO] Building lab06-api 1.0.0                                           [2/3]
[INFO] Compiling 5 source files with javac [debug deprecation release 21] to target/classes
[INFO] Compiling 1 source file with javac [debug deprecation release 21] to target/test-classes
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO] Building lab06-consumer 1.0.0                                      [3/3]
[INFO] Compiling 1 source file with javac [debug deprecation release 21] to target/classes
[ERROR] COMPILATION ERROR :
[ERROR] consumer/src/main/java/edu/cmu/cs214/frontdesk/FrontDesk.java:[27,19] method createBooking in interface edu.cmu.cs214.booking.BookingApi cannot be applied to given types;
  required: edu.cmu.cs214.booking.BookingRequest
  found:    java.lang.String,long,long,<nulltype>
  reason: actual and formal argument lists differ in length
[ERROR] consumer/src/main/java/edu/cmu/cs214/frontdesk/FrontDesk.java:[33,19] method createBooking in interface edu.cmu.cs214.booking.BookingApi cannot be applied to given types;
  required: edu.cmu.cs214.booking.BookingRequest
  found:    java.lang.String,long,long,java.lang.String
  reason: actual and formal argument lists differ in length
[INFO] 2 errors
[INFO] Reactor Summary for lab06-booking-parent 1.0.0:
[INFO] lab06-booking-parent ............................... SUCCESS
[INFO] lab06-api .......................................... SUCCESS
[INFO] lab06-consumer ..................................... FAILURE
[INFO] BUILD FAILURE
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.13.0:compile (default-compile) on project lab06-consumer: Compilation failure
```

**Which module's tests ran, and which did not.** And what that tells you about
who can detect a contract break.

The api module's 5 tests ran and passed. The consumer's 7 tests never ran:
Maven stopped `lab06-consumer` at the compile step on `FrontDesk.java:27` and
`:33`, before reaching the test phase. So a green api suite says nothing about
compatibility. I rewrote those tests to the new call, so they only prove the
new method works. Only the consumer's own build, code I don't own, can detect
that something it relied on was removed.

### Step 2: the deprecation path

**What you added.** The signatures that came back, and what they delegate to.

Both came back on `BookingApi` as `@Deprecated` `default` methods, so
implementations only have to provide the new method:

```java
@Deprecated
default Booking createBooking(String roomId, long startMinute, long endMinute,
                              String waitlistKey)
    // -> createBooking(BookingRequest.of(roomId, startMinute, endMinute)
    //                      .withWaitlistKey(waitlistKey))

@Deprecated
default Booking createBooking(String roomId, long startMinute, long endMinute,
                              String waitlistKey, String notes)
    // -> createBooking(BookingRequest.of(roomId, startMinute, endMinute)
    //                      .withWaitlistKey(waitlistKey).withNotes(notes))
```

Each has a `@deprecated` javadoc tag naming `createBooking(BookingRequest)` as
the replacement.

**The warnings.** Paste one deprecation warning line from the build log (from
a `mvn -B clean test` run, since a rerun with nothing to compile prints none).

`mvn -B clean test` (absolute path prefix trimmed):

```
[INFO] Building lab06-api 1.0.0                                           [2/3]
[INFO] Compiling 5 source files with javac [debug deprecation release 21] to target/classes
[INFO] Compiling 1 source file with javac [debug deprecation release 21] to target/test-classes
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO] Building lab06-consumer 1.0.0                                      [3/3]
[INFO] Compiling 1 source file with javac [debug deprecation release 21] to target/classes
[WARNING] consumer/src/main/java/edu/cmu/cs214/frontdesk/FrontDesk.java:[27,19] createBooking(java.lang.String,long,long,java.lang.String) in edu.cmu.cs214.booking.BookingApi has been deprecated
[WARNING] consumer/src/main/java/edu/cmu/cs214/frontdesk/FrontDesk.java:[33,19] createBooking(java.lang.String,long,long,java.lang.String) in edu.cmu.cs214.booking.BookingApi has been deprecated
[INFO] Compiling 1 source file with javac [debug deprecation release 21] to target/test-classes
[INFO] Tests run: 7, Failures: 0, Errors: 0, Skipped: 0
[INFO] lab06-booking-parent ............................... SUCCESS
[INFO] lab06-api .......................................... SUCCESS
[INFO] lab06-consumer ..................................... SUCCESS
[INFO] BUILD SUCCESS
```

**What the deprecation path resolves.** Who can now build that could not build
during step 1, and who is on which schedule.

After step 1, the untouched front desk app could not build at all; its only
option was rewriting `FrontDesk.java` on the day the API changed. After step 2
it builds and all 7 tests pass unchanged, while new code (and the api tests)
use `createBooking(BookingRequest)`. Schedules: the API team ships the new
method now. The front desk team migrates its two call sites whenever it
chooses during the deprecation window. The old overloads are removed only in a
later, announced release, after callers have moved.

**What the warnings accomplish that a README note would not.** Be concrete
about where the warning shows up and who sees it without looking for it.

The warning shows up in the front desk team's own build log on every compile,
naming the exact file, line, and column (`FrontDesk.java:[27,19]` and
`[33,19]`), and their IDE flags the same calls. They see it without looking
for it, while a README note only reaches someone who goes and reads our docs.
The `@deprecated` javadoc names the replacement right where they are working,
and when the warnings stop appearing, the migration is done.

---

## Milestone 3: The misuse critique

Not coded. One misuse, one redesign, one cost. Discuss it with your TA.

### The misuse

**What is easy to get wrong.** One specific thing about the API surface.

The `createBooking` overloads take `waitlistKey` and `notes` as plain
`String`s. Java picks an overload at compile time by argument count and
static types, so the compiler can't tell which meaning a caller intended: a
call binds to the overload whose types match, which may not be the one the
caller meant. The 4-arg and 5-arg overloads differ only by a trailing `String`,
so dropping an argument or swapping two compiles without complaint.

**The call site.** File and line in `consumer/`, with the call. Show the
code that a reader cannot understand without opening the javadoc, or that a
caller could get wrong with the compiler still happy.

`FrontDesk.java:33`:

```java
return api.createBooking(roomId, startMinute, endMinute, guestName);
```

Nothing on this line says the guest name is a waitlist key, or that this
argument changes what happens when the room is busy. Since M1 there is also a
5-arg overload with notes last. A developer who wants to note "late checkout"
on a walk-in writes `api.createBooking(roomId, s, e, "late checkout")`. It
compiles, binds to the 4-arg overload, and the note becomes the waitlist key.

**What goes wrong when it happens.** Silent bad behavior, wrong data, a crash
somewhere far away?

Silent wrong behavior, no exception (checked by running it against this
build). `getNotes()` is null, so the note is lost. On a free room the
schedule shows `CONFIRMED (late checkout)`. On a busy room, a walk-in that
should be turned away (null) is WAITLISTED instead, and a later
`cancelAndOfferToWaitlist` (`FrontDesk.java:48`) promotes it to CONFIRMED,
holding a room for a guest who already left. The bug surfaces far from where
it was written.

### The redesign

**The proposal.** Types, enums, factories, or whatever you are proposing. Show
the new signature and the new call site.

Give the conflict behavior its own type instead of a `String` with a null
sentinel, and remove the positional `String` overloads after the deprecation
window:

```java
public sealed interface OnConflict {
    record TurnAway() implements OnConflict {}
    record Waitlist(String key) implements OnConflict {
        public Waitlist { Objects.requireNonNull(key); }
    }
}

// BookingRequest holds an OnConflict instead of a String key:
public BookingRequest onConflict(OnConflict policy)
public BookingRequest withNotes(String notes)
```

New call sites:

```java
// FrontDesk.java:27
api.createBooking(BookingRequest.of(roomId, s, e).onConflict(new OnConflict.TurnAway()));
// FrontDesk.java:33
api.createBooking(BookingRequest.of(roomId, s, e)
        .onConflict(new OnConflict.Waitlist(guestName)));
```

**Why the mistake is now hard or impossible to make.** Point at the mechanism,
such as the compiler, a validating constructor, or an exhaustive switch.

The compiler. `onConflict` takes an `OnConflict`, so passing a note there is
a compile error (`incompatible types: String cannot be converted to
OnConflict`, checked with javac). Notes have exactly one place to go,
`withNotes(String)`. "Turn away" is a named value instead of `null`, and the
`Waitlist` record's constructor rejects a null key. Not every `String` needs
wrapping; this targets the one parameter whose value changes behavior.

### One tradeoff

**What it costs.** Something real, such as caller ceremony, migration burden
against the deprecation path you just built, or more types for a newcomer to
learn. "No real downside" does not count.

Another migration for a team we don't control. The front desk team was just
pointed at `BookingRequest` by the deprecation warnings. This changes the
request again, so the same two call sites (`:27`, `:33`) change twice, and we
run a second deprecation cycle. A smaller cost: a one-line call becomes a
builder chain with more types for a newcomer to learn.

**When the price is worth paying.** A condition under which it is.

When the mistake is silent and costly (a room held for a guest who left) and
the callers are teams whose code we can't review, so the compiler has to catch
it for us. Ship it in the same release that removes the deprecated overloads,
so the front desk team migrates once instead of twice.
