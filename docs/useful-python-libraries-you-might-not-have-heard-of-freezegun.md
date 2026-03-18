# 你可能没听过的有用 Python 库：Freezegun

> 原文：[`towardsdatascience.com/useful-python-libraries-you-might-not-have-heard-of-freezegun/`](https://towardsdatascience.com/useful-python-libraries-you-might-not-have-heard-of-freezegun/)

<mdspan datatext="el1756945676472" class="mdspan-comment">我认为我们都可以同意</mdspan>，测试我们的代码是软件开发生命周期中一个关键且必要的部分。当我们讨论 AI 和 ML 系统时，这可能更为重要，因为固有的不确定性和幻觉元素可能从一开始就已经嵌入。

在这个通用测试框架内，测试基于当前日期或时间表现不同的代码可能真的让人头疼。你如何可靠地检查仅在午夜触发的逻辑、计算相对日期（“2 小时前”）或处理像闰年或月末这样的棘手情况？手动模拟 Python 的 datetime 模块可能会很繁琐且容易出错。

如果你曾经为此而挣扎过，你并不孤单。但如果你能简单地……停止时间？或者甚至在你测试中穿越时间呢？

这正是 Freezegun 库让你做到的。它是对一个常见测试问题的优雅解决方案，但许多经验丰富的 Python 开发者从未听说过它。

**Freezegun** 允许你的 Python 测试通过模拟 datetime、date、time 和 pendulum Python 模块来模拟特定的时间点。它使用简单但功能强大，可以创建针对时间敏感代码的确定性和可靠的测试。

## 为什么 Freezegun 如此有用？

1.  **确定性**。这是 Freezegun 的主要优势。涉及时间的测试变得完全可预测。在冻结块内运行 datetime.now()返回相同的冻结时间戳，消除了由于测试执行期间毫秒差异或日期回滚而导致的不可靠测试。

1.  **简单性**。与手动修补 **datetime.now** 或使用 **unittest.mock** 相比，Freezegun 通常更干净，且需要的样板代码更少，尤其是在临时更改时间时。

1.  **时间旅行**。轻松模拟特定日期和时间——过去、现在或未来。这对于测试边缘情况至关重要，例如年终处理、闰秒、夏令时转换，或者简单地验证与特定事件相关的逻辑。

1.  **相对时间测试**。通过冻结时间并创建相对于该冻结时刻的时间戳来测试计算相对时间的函数（例如，“3 天后到期”）。

1.  **滴答声**。Freezegun 允许时间在测试中从冻结的瞬间前进（“滴答”），这对于测试超时、持续时间或时间依赖事件的序列来说非常完美。

希望我已经说服你，Freezegun 可以成为你的 Python 工具箱中的一个宝贵补充。让我们通过查看一些示例代码片段来实际看看它的作用。

## 设置开发环境

在此之前，让我们设置一个开发环境来实验。我使用 Miniconda，但你可以使用你熟悉的任何工具。

我是一个 Windows 用户，但经常使用 WSL2 Ubuntu for Windows 进行开发，这就是我将在这里做的事情。

我展示的所有代码在 Windows 或类 Unix 操作系统下都应该能正常工作。

```py
# Create and activate a new dev environment
#
(base) $ conda create -n freezegun python=3.12 -y
(base) $ conda activate freezegun
```

现在，我们可以安装剩余的必要库。

```py
(freezegun) $ pip install freezegun jupyter
```

我将使用 Jupyter Notebook 来运行我的代码。要跟随，请在命令提示符中输入`jupyter notebook`。你应该在浏览器中看到一个 jupyter notebook 打开。如果这没有自动发生，你可能会在`jupyter notebook`命令后看到一屏的信息。在底部附近，你会找到一个 URL，你可以复制并粘贴到浏览器中启动 Jupyter Notebook。

你的 URL 将与我不同，但它应该看起来像这样：-

```py
http://127.0.0.1:8888/tree?token=3b9f7bd07b6966b41b68e2350721b2d0b6f388d248cc69da
```

> 简单地说：以下示例中展示的代码大量使用了 Python 的**assert**命令。如果你之前没有遇到过这个函数，或者没有在 Python 中进行过很多单元测试，assert 用于测试一个条件是否为真，如果不是，则抛出`AssertionError`。这有助于在开发过程中捕捉问题，并且通常用于调试和验证代码中的假设。

### 示例 1：使用装饰器进行基本时间冻结

使用 Freezegun 最常见的方式是通过其装饰器**@freeze_time**，它允许你“设置”一天中的特定时间来测试各种与时间相关的函数。

```py
import datetime
from freezegun import freeze_time

def get_greeting():
    now = datetime.datetime.now()
    print(f"  Inside get_greeting(), now = {now}") # Added print
    if now.hour < 12:
        return "Good morning!"
    elif 12 <= now.hour < 18:
        return "Good afternoon!"
    else:
        return "Good evening!"

# Test the morning greeting
@freeze_time("2023-10-27 09:00:00")
def test_morning_greeting():
    print("Running test_morning_greeting:")
    greeting = get_greeting()
    print(f"  -> Got greeting: '{greeting}'")
    assert greeting == "Good morning!"

# Test the evening greeting
@freeze_time("2023-10-27 21:30:00")
def test_evening_greeting():
    print("\nRunning test_evening_greeting:")
    greeting = get_greeting()
    print(f"  -> Got greeting: '{greeting}'")
    assert greeting == "Good evening!"

# Run the tests
test_morning_greeting()
test_evening_greeting()
print("\nBasic decorator tests passed!")

# --- Failure Scenario ---
# What happens if we don't freeze time?
print("\n--- Running without freeze_time (might fail depending on actual time) ---")
def test_morning_greeting_unfrozen():
    print("Running test_morning_greeting_unfrozen:")
    greeting = get_greeting()
    print(f"  -> Got greeting: '{greeting}'")
    # This assertion is now unreliable! It depends on when you run the code.
    try:
        assert greeting == "Good morning!" 
        print("  (Passed by chance)")
    except AssertionError:
        print("  (Failed as expected - time wasn't 9 AM)")

test_morning_greeting_unfrozen()
```

以及输出结果。

```py
Running test_morning_greeting:
  Inside get_greeting(), now = 2023-10-27 09:00:00
  -> Got greeting: 'Good morning!'

Running test_evening_greeting:
  Inside get_greeting(), now = 2023-10-27 21:30:00
  -> Got greeting: 'Good evening!'

Basic decorator tests passed!

--- Running without freeze_time (might fail depending on actual time) ---
Running test_morning_greeting_unfrozen:
  Inside get_greeting(), now = 2025-04-16 15:00:37.363367
  -> Got greeting: 'Good afternoon!'
  (Failed as expected - time wasn't 9 AM)
```

### 示例 2：使用上下文管理器进行基本时间冻结

创建一个“冻结时间”的“块”。

```py
import datetime
from freezegun import freeze_time

def process_batch_job():
    start_time = datetime.datetime.now()
    # Simulate work
    end_time = datetime.datetime.now() # In reality, time would pass
    print(f"  Inside job: Start={start_time}, End={end_time}") # Added print
    return (start_time, end_time)

def test_job_timestamps_within_frozen_block():
    print("\nRunning test_job_timestamps_within_frozen_block:")
    frozen_time_str = "2023-11-15 10:00:00"
    with freeze_time(frozen_time_str):
        print(f"  Entering frozen block at {frozen_time_str}")
        start, end = process_batch_job()

        print(f"  Asserting start == end: {start} == {end}")
        assert start == end
        print(f"  Asserting start == frozen time: {start} == {datetime.datetime(2023, 11, 15, 10, 0, 0)}")
        assert start == datetime.datetime(2023, 11, 15, 10, 0, 0)
        print("  Assertions inside block passed.")

    print("  Exited frozen block.")
    now_outside = datetime.datetime.now()
    print(f"  Time outside block: {now_outside} (should be real time)")
    # This assertion just shows time is unfrozen, value depends on real time
    assert now_outside != datetime.datetime(2023, 11, 15, 10, 0, 0)

test_job_timestamps_within_frozen_block()
print("\nContext manager test passed!")
```

输出结果。

```py
 Running test_job_timestamps_within_frozen_block:
 Entering frozen block at 2023-11-15 10:00:00
 Inside job: Start=2023-11-15 10:00:00, End=2023-11-15 10:00:00
 Asserting start == end: 2023-11-15 10:00:00 == 2023-11-15 10:00:00
 Asserting start == frozen time: 2023-11-15 10:00:00 == 2023-11-15 10:00:00
 Assertions inside block passed.
 Exited frozen block.
 Time outside block: 2025-04-16 15:10:15.231632 (should be real time)

 Context manager test passed!
```

### 示例 3：使用 tick 前进时间

在冻结期间模拟时间的流逝。

```py
import datetime
import time
from freezegun import freeze_time

def check_if_event_expired(event_timestamp, expiry_duration_seconds):
    now = datetime.datetime.now()
    expired = now > event_timestamp + datetime.timedelta(seconds=expiry_duration_seconds)
    print(f"  Checking expiry: Now={now}, Event={event_timestamp}, ExpiresAt={event_timestamp + datetime.timedelta(seconds=expiry_duration_seconds)} -> Expired={expired}")
    return expired

# --- Manual ticking using context manager ---
def test_event_expiry_manual_tick():
    print("\nRunning test_event_expiry_manual_tick:")

    with freeze_time("2023-10-27 12:00:00") as freezer:
        event_time_in_freeze = datetime.datetime.now()
        expiry_duration = 60
        print(f"  Event created at: {event_time_in_freeze}")

        print("  Checking immediately after creation:")
        assert not check_if_event_expired(event_time_in_freeze, expiry_duration)

        # Advance time by 61 seconds
        delta_to_tick = datetime.timedelta(seconds=61)
        print(f"  Ticking forward by {delta_to_tick}...")
        freezer.tick(delta=delta_to_tick)

        print(f"  Time after ticking: {datetime.datetime.now()}")
        print("  Checking after ticking:")
        assert check_if_event_expired(event_time_in_freeze, expiry_duration)

        print("  Manual tick test finished.")

# --- Failure Scenario ---
@freeze_time("2023-10-27 12:00:00")  # No tick=True or manual tick
def test_event_expiry_fail_without_tick():
    print("\n--- Running test_event_expiry_fail_without_tick (EXPECT ASSERTION ERROR) ---")
    event_time = datetime.datetime.now()
    expiry_duration = 60
    print(f"  Event created at: {event_time}")

    # Simulate work or waiting - without tick, time doesn't advance!
    time.sleep(0.1)

    print(f"  Time after simulated wait: {datetime.datetime.now()}")
    print("  Checking expiry (incorrectly, time didn't move):")
    try:
        # This should ideally be True, but will be False without ticking
        assert check_if_event_expired(event_time, expiry_duration)
    except AssertionError:
        print("  AssertionError: Event did not expire, as expected without tick.")
    print("  Failure scenario finished.")

# Run both tests
test_event_expiry_manual_tick()
test_event_expiry_fail_without_tick()
```

这将输出以下内容。

```py
Running test_event_expiry_manual_tick:
  Event created at: 2023-10-27 12:00:00
  Checking immediately after creation:
  Checking expiry: Now=2023-10-27 12:00:00, Event=2023-10-27 12:00:00, ExpiresAt=2023-10-27 12:01:00 -> Expired=False
  Ticking forward by 0:01:01...
  Time after ticking: 2023-10-27 12:01:01
  Checking after ticking:
  Checking expiry: Now=2023-10-27 12:01:01, Event=2023-10-27 12:00:00, ExpiresAt=2023-10-27 12:01:00 -> Expired=True
  Manual tick test finished.

--- Running test_event_expiry_fail_without_tick (EXPECT ASSERTION ERROR) ---
  Event created at: 2023-10-27 12:00:00
  Time after simulated wait: 2023-10-27 12:00:00
  Checking expiry (incorrectly, time didn't move):
  Checking expiry: Now=2023-10-27 12:00:00, Event=2023-10-27 12:00:00, ExpiresAt=2023-10-27 12:01:00 -> Expired=False
  AssertionError: Event did not expire, as expected without tick.
  Failure scenario finished.
```

### 示例 4：测试相对日期

Freezegun 确保“时间过去”逻辑的稳定性。

```py
import datetime
from freezegun import freeze_time

def format_relative_time(timestamp):
    now = datetime.datetime.now()
    delta = now - timestamp

    rel_time_str = ""
    if delta.days > 0:
        rel_time_str = f"{delta.days} days ago"
    elif delta.seconds >= 3600:
        hours = delta.seconds // 3600
        rel_time_str = f"{hours} hours ago"
    elif delta.seconds >= 60:
        minutes = delta.seconds // 60
        rel_time_str = f"{minutes} minutes ago"
    else:
        rel_time_str = "just now"
    print(f"  Formatting relative time: Now={now}, Timestamp={timestamp} -> '{rel_time_str}'")
    return rel_time_str

@freeze_time("2023-10-27 15:00:00")
def test_relative_time_formatting():
    print("\nRunning test_relative_time_formatting:")

    # Event happened 2 days and 3 hours ago relative to frozen time
    past_event = datetime.datetime(2023, 10, 25, 12, 0, 0)
    assert format_relative_time(past_event) == "2 days ago"

    # Event happened 45 minutes ago
    recent_event = datetime.datetime.now() - datetime.timedelta(minutes=45)
    assert format_relative_time(recent_event) == "45 minutes ago"

    # Event happened just now
    current_event = datetime.datetime.now() - datetime.timedelta(seconds=10)
    assert format_relative_time(current_event) == "just now"

    print("  Relative time tests passed!")

test_relative_time_formatting()

# --- Failure Scenario ---
print("\n--- Running relative time without freeze_time (EXPECT FAILURE) ---")
def test_relative_time_unfrozen():
    # Use the same past event timestamp
    past_event = datetime.datetime(2023, 10, 25, 12, 0, 0) 
    print(f"  Testing with past_event = {past_event}")
    # This will compare against the *actual* current time, not Oct 27th, 2023
    formatted_time = format_relative_time(past_event)
    try:
        assert formatted_time == "2 days ago" 
    except AssertionError:
        # The actual difference will be much larger!
        print(f"  AssertionError: Expected '2 days ago', but got '{formatted_time}'. Failed as expected.")

test_relative_time_unfrozen()
```

输出结果。

```py
Running test_relative_time_formatting:
  Formatting relative time: Now=2023-10-27 15:00:00, Timestamp=2023-10-25 12:00:00 -> '2 days ago'
  Formatting relative time: Now=2023-10-27 15:00:00, Timestamp=2023-10-27 14:15:00 -> '45 minutes ago'
  Formatting relative time: Now=2023-10-27 15:00:00, Timestamp=2023-10-27 14:59:50 -> 'just now'
  Relative time tests passed!

--- Running relative time without freeze_time (EXPECT FAILURE) ---
  Testing with past_event = 2023-10-25 12:00:00
  Formatting relative time: Now=2023-10-27 12:00:00, Timestamp=2023-10-25 12:00:00 -> '2 days ago'
```

### 示例 5：处理特定日期（月末）

可靠地测试边缘情况，例如闰年。

```py
import datetime
from freezegun import freeze_time

def is_last_day_of_month(check_date):
    next_day = check_date + datetime.timedelta(days=1)
    is_last = next_day.month != check_date.month
    print(f"  Checking if {check_date} is last day of month: Next day={next_day}, IsLast={is_last}")
    return is_last

print("\nRunning specific date logic tests:")

@freeze_time("2023-02-28") # Non-leap year
def test_end_of_february_non_leap():
    today = datetime.date.today()
    assert is_last_day_of_month(today) is True

@freeze_time("2024-02-28") # Leap year
def test_end_of_february_leap_not_yet():
     today = datetime.date.today()
     assert is_last_day_of_month(today) is False # Feb 29th exists

@freeze_time("2024-02-29") # Leap year - last day
def test_end_of_february_leap_actual():
    today = datetime.date.today()
    assert is_last_day_of_month(today) is True

@freeze_time("2023-12-31")
def test_end_of_year():
    today = datetime.date.today()
    assert is_last_day_of_month(today) is True

test_end_of_february_non_leap()
test_end_of_february_leap_not_yet()
test_end_of_february_leap_actual()
test_end_of_year()
print("Specific date logic tests passed!")

#
# Output
#

Running specific date logic tests:
Checking if 2023-02-28 is last day of month: Next day=2023-03-01, IsLast=True
Checking if 2024-02-28 is last day of month: Next day=2024-02-29, IsLast=False
Checking if 2024-02-29 is last day of month: Next day=2024-03-01, IsLast=True
Checking if 2023-12-31 is last day of month: Next day=2024-01-01, IsLast=True
pecific date logic tests passed!
```

### 示例 6：时区

正确测试时区感知代码，处理 BST/GMT 等偏移和转换。

```py
# Requires Python 3.9+ for zoneinfo or `pip install pytz` for older versions
import datetime
from freezegun import freeze_time
try:
    from zoneinfo import ZoneInfo # Python 3.9+
except ImportError:
    from pytz import timezone as ZoneInfo # Fallback for older Python/pytz

def get_local_and_utc_time():
    # Assume local timezone is Europe/London for this example
    local_tz = ZoneInfo("Europe/London")
    now_utc = datetime.datetime.now(datetime.timezone.utc)
    now_local = now_utc.astimezone(local_tz)
    print(f"  Getting times: UTC={now_utc}, Local={now_local} ({now_local.tzname()})")
    return now_local, now_utc

# Freeze time as 9 AM UTC. London is UTC+1 in summer (BST). Oct 27 is BST.
@freeze_time("2023-10-27 09:00:00", tz_offset=0) # tz_offset=0 means the frozen time string IS UTC
def test_time_in_london_bst():
    print("\nRunning test_time_in_london_bst:")
    local_time, utc_time = get_local_and_utc_time()
    assert utc_time.hour == 9
    assert local_time.hour == 10 # London is UTC+1 on this date
    assert local_time.tzname() == "BST" 

# Freeze time as 9 AM UTC. Use December 27th, which is GMT (UTC+0)
@freeze_time("2023-12-27 09:00:00", tz_offset=0)
def test_time_in_london_gmt():
    print("\nRunning test_time_in_london_gmt:")
    local_time, utc_time = get_local_and_utc_time()
    assert utc_time.hour == 9
    assert local_time.hour == 9 # London is UTC+0 on this date
    assert local_time.tzname() == "GMT"

test_time_in_london_bst()
test_time_in_london_gmt()
print("\nTimezone tests passed!")

#
# Output
#

 Running test_time_in_london_bst:
 Getting times: UTC=2023-10-27 09:00:00+00:00, Local=2023-10-27 10:00:00+01:00 (BST)

 Running test_time_in_london_gmt:
 Getting times: UTC=2023-12-27 09:00:00+00:00, Local=2023-12-27 09:00:00+00:00 (GMT)

 Timezone tests passed!
```

### 示例 7：使用 move_to 函数进行显式时间穿越

在单个测试中跳转到特定的时间点，以处理复杂的时序。

```py
import datetime
from freezegun import freeze_time

class ReportGenerator:
    def __init__(self):
        self.creation_time = datetime.datetime.now()
        self.data = {"status": "pending", "generated_at": None}
        print(f"  Report created at {self.creation_time}")

    def generate(self):
        self.data["status"] = "generated"
        self.data["generated_at"] = datetime.datetime.now()
        print(f"  Report generated at {self.data['generated_at']}")

    def get_status_update(self):
        now = datetime.datetime.now()
        if self.data["status"] == "generated":
            time_since_generation = now - self.data["generated_at"]
            status = f"Generated {time_since_generation.seconds} seconds ago."
        else:
            time_since_creation = now - self.creation_time
            status = f"Pending for {time_since_creation.seconds} seconds."
        print(f"  Status update at {now}: '{status}'")
        return status

def test_report_lifecycle():
    print("\nRunning test_report_lifecycle:")
    with freeze_time("2023-11-01 10:00:00") as freezer:
        report = ReportGenerator()
        assert report.data["status"] == "pending"

        # Check status after 5 seconds
        target_time = datetime.datetime(2023, 11, 1, 10, 0, 5)
        print(f"  Moving time to {target_time}")
        freezer.move_to(target_time)
        assert report.get_status_update() == "Pending for 5 seconds."

        # Generate the report at 10:01:00
        target_time = datetime.datetime(2023, 11, 1, 10, 1, 0)
        print(f"  Moving time to {target_time} and generating report")
        freezer.move_to(target_time)
        report.generate()
        assert report.data["status"] == "generated"
        assert report.get_status_update() == "Generated 0 seconds ago."

        # Check status 30 seconds after generation
        target_time = datetime.datetime(2023, 11, 1, 10, 1, 30)
        print(f"  Moving time to {target_time}")
        freezer.move_to(target_time)
        assert report.get_status_update() == "Generated 30 seconds ago."

    print("  Complex lifecycle test passed!")

test_report_lifecycle()

# --- Failure Scenario ---
def test_report_lifecycle_fail_forgot_move():
    print("\n--- Running lifecycle test (FAIL - forgot move_to) ---")
    with freeze_time("2023-11-01 10:00:00") as freezer:
        report = ReportGenerator()
        assert report.data["status"] == "pending"

        # We INTEND to check status after 5 seconds, but FORGET to move time
        print(f"  Checking status (time is still {datetime.datetime.now()})")
        # freezer.move_to("2023-11-01 10:00:05") # <-- Forgotten!
        try:
            assert report.get_status_update() == "Pending for 5 seconds."
        except AssertionError as e:
            print(f"  AssertionError: {e}. Failed as expected.")

test_report_lifecycle_fail_forgot_move()
```

这里是输出结果。

```py
Running test_report_lifecycle:
  Report created at 2023-11-01 10:00:00
  Moving time to 2023-11-01 10:00:05
  Status update at 2023-11-01 10:00:05: 'Pending for 5 seconds.'
  Moving time to 2023-11-01 10:01:00 and generating report
  Report generated at 2023-11-01 10:01:00
  Status update at 2023-11-01 10:01:00: 'Generated 0 seconds ago.'
  Moving time to 2023-11-01 10:01:30
  Status update at 2023-11-01 10:01:30: 'Generated 30 seconds ago.'
  Complex lifecycle test passed!

--- Running lifecycle test (FAIL - forgot move_to) ---
  Report created at 2023-11-01 10:00:00
  Checking status (time is still 2023-11-01 10:00:00)
  Status update at 2023-11-01 10:00:00: 'Pending for 0 seconds.'
  AssertionError: . Failed as expected.
```

## 摘要

Freezegun 是任何需要测试涉及日期和时间的 Python 开发者的绝佳工具。它将可能不稳定的、难以编写的测试转换为简单、健壮和确定的测试。通过允许你轻松地冻结、滴答和穿越时间，并且清楚地表明时间*没有被控制*，它解锁了有效地和可靠地测试以前具有挑战性的场景的能力。

为了说明这一点，我提供了几个涵盖不同实例的例子，这些实例涉及日期和时间测试，并展示了如何使用 Freezegun 消除传统测试框架可能遇到的许多障碍。

虽然我们已经涵盖了核心功能，但你还可以用 Freezegun 做更多的事情，我建议查看其[GitHub 页面](https://github.com/spulec/freezegun)。

简而言之，Freezegun 是一个你应该了解并使用的库，如果你的代码处理时间并且需要彻底且可靠地进行测试。
