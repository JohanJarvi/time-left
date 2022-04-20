<script>
  const convertTimeToMinutes = (time) => {
    const hourAndMinutes = time.split(":");
    const hour = Number(hourAndMinutes[0]);
    const minute = Number(hourAndMinutes[1]);

    return hour * 60 + minute;
  };

  const convertMinutesToTime = (minutes) => {
    // if (minutes < 0) return `0:00`;
    const hourPortion = Math.floor(minutes / 60);
    const minutePortion = Math.round(((minutes / 60) % 1) * 60);

    if (minutePortion === 60) {
      return `${hourPortion + 1} hours`;
    }

    return `${hourPortion} hours and ${minutePortion} minutes`;
  };

  const getTimeLeftToWorkMessage = (timeLeft) => {
    if (timeLeft === "0:00")
      return "You have worked too much already! Stop working this instant!";

    if (timeLeft.includes("NaN")) {
      return `If today is day one then see time left today below, otherwise enter times worked above.`;
    }

    return `You have ${timeLeft} left to work this week.`;
  };

  const getTimeLeftToWorkTodayMessage = (timeLeft) => {
    if (timeLeft.includes("NaN")) {
      return `Please use the date time picker to select the time started today!`;
    }
    return `You have ${timeLeft} left to work today.`;
  };

  const getMinutesTotalForWorkDays = (workDays) => {
    return Math.ceil(7.6 * workDays * 60);
  };

  // Inputs
  let timeStartedTodayInput = "";
  let timesWorkedThisWeekInput = "0:00";
  let amountOfBreakInput = "0:00";
  let daysWorked = 0;

  // Global constants
  $: minutesInWorkWeek = getMinutesTotalForWorkDays(daysWorked);

  // Global variables
  let timeLeftToWorkMessage = "Click the button to update the times left";
  let timeLeftToWorkTodayMessage = "";
  let timeBetweenStartedAndNowInMinutes = 0;
  let workedToday = 0;

  const getMinutesRemainingToday = (timeStartedToday) => {
    const startedToday = new Date(timeStartedToday);
    const timeNow = new Date();
    const timeBetweenStartedAndNowInMilliseconds = Math.abs(
      startedToday - timeNow
    );

    timeBetweenStartedAndNowInMinutes = Math.floor(
      timeBetweenStartedAndNowInMilliseconds / 1000 / 60
    );

    const timeWorkedTodayMinutes = timeBetweenStartedAndNowInMinutes;
    const timeOfWorkDayInMinutes = convertTimeToMinutes("7:36");
    const timeLeftMinutes = timeOfWorkDayInMinutes - timeWorkedTodayMinutes;

    return timeLeftMinutes;
  };

  const getRemainingMinutesToWorkInWeek = (
    timeWorkedThisWeekInput,
    minutesInWorkWeek
  ) => {
    if (!timeWorkedThisWeekInput) return 0;
    const timeWorked = timeWorkedThisWeekInput.split(",");

    const timeWorkedInMinutes = timeWorked.map((time) =>
      convertTimeToMinutes(time)
    );

    const totalTimeWorkedInMinutes = timeWorkedInMinutes.reduce(
      (a, b) => a + b
    );
    return minutesInWorkWeek - totalTimeWorkedInMinutes;
  };

  let minutesRemainingToday = 0;

  const updateMessage = (
    timeStartedToday,
    timesWorkedThisWeek,
    amountOfBreak,
    timeNow,
    minutesInWorkWeek
  ) => {
    const minutesOfBreak = convertTimeToMinutes(amountOfBreak);
    const minutesRemainingInWorkWeek =
      getRemainingMinutesToWorkInWeek(timesWorkedThisWeek, minutesInWorkWeek) +
      minutesOfBreak;
    minutesRemainingToday =
      getMinutesRemainingToday(timeStartedToday) + minutesOfBreak;
    const timeLeft = convertMinutesToTime(
      minutesRemainingInWorkWeek - timeBetweenStartedAndNowInMinutes
    );
    const timeLeftToday = convertMinutesToTime(minutesRemainingToday);
    workedToday = convertMinutesToTime(
      timeBetweenStartedAndNowInMinutes - minutesOfBreak
    );
    if (
      minutesRemainingInWorkWeek - timeBetweenStartedAndNowInMinutes <
      minutesRemainingToday
    ) {
      timeLeftToWorkTodayMessage = "";
    } else {
      timeLeftToWorkTodayMessage = getTimeLeftToWorkTodayMessage(timeLeftToday);
    }

    timeLeftToWorkMessage = getTimeLeftToWorkMessage(timeLeft);
  };

  let timeNow = new Date();
  setInterval(() => {
    timeNow = new Date();
  }, 1000);

  // Reactive declarations
  $: updateMessage(
    timeStartedTodayInput,
    timesWorkedThisWeekInput,
    amountOfBreakInput,
    timeNow,
    minutesInWorkWeek
  );

  const getFinishingTime = (timeNow, minutesRemainingToday) => {
    return new Date(timeNow.getTime() + minutesRemainingToday * 60000);
  };

  $: finishTime = getFinishingTime(timeNow, minutesRemainingToday);
</script>

<main>
  <h1>Time left</h1>
  <p>How many days will you work this week?</p>
  <select bind:value={daysWorked}>
    <option>1</option>
    <option>2</option>
    <option>3</option>
    <option>4</option>
    <option>5</option>
    <option>6</option>
    <option>7</option>
  </select>
  <p>
    Please enter the times you have worked this week as a comma seperated list
    (e.g. 8:12, 4:54, ..., 5:57):
  </p>
  <input bind:value={timesWorkedThisWeekInput} />
  <p>Please select the time you started today (e.g. 7:45):</p>
  <input type="datetime-local" bind:value={timeStartedTodayInput} />
  <p>Please enter the amount of break time you have had today (e.g. 0:45):</p>
  <input bind:value={amountOfBreakInput} />
  <p>
    <strong>
      {timeLeftToWorkMessage}
    </strong>
  </p>
  <p>
    <strong>
      {timeLeftToWorkTodayMessage}
    </strong>
  </p>
  {#if !workedToday.includes("NaN")}
    <p>
      You've worked {workedToday} today.
    </p>
  {/if}
  <div class="container">
    <h2>Time Now</h2>
    <p>
      <strong>
        {timeNow.toLocaleTimeString()}
      </strong>
    </p>
    <p>
      {finishTime == "Invalid Date"
        ? ""
        : "You will finish your current work day at:"}
    </p>
    <p>
      <strong>
        {finishTime == "Invalid Date" ? "" : finishTime.toLocaleString()}
      </strong>
    </p>
  </div>
</main>

<style>
  main {
    padding-left: 100px;
  }

  h1 {
    color: #ff3e00;
    text-transform: capitalize;
    font-size: 4em;
    font-weight: 100;
  }

  .container {
    margin-top: 50px;
  }
</style>
