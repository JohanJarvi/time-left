<script>
  const convertTimeToMinutes = (time) => {
    const hourAndMinutes = time.split(":");
    const hour = Number(hourAndMinutes[0]);
    const minute = Number(hourAndMinutes[1]);

    return hour * 60 + minute;
  };

  const convertMinutesToTime = (minutes, isWorkedToday = false) => {
    let hourPortion = Math.floor(minutes / 60);
    const minutePortion = Math.round(((minutes / 60) % 1) * 60);

    if (minutePortion === 60) {
      return `${hourPortion + 1} hours`;
    }

    if (isWorkedToday) {
      return `${hourPortion} hours and ${minutePortion} minutes`;
    }

    if (minutes < 0) {
      hourPortion += 1;
      return `worked ${Math.abs(hourPortion)} hours and ${Math.abs(
        minutePortion
      )} minutes too much`;
    }

    return `${hourPortion} hours and ${minutePortion} minutes left to work`;
  };

  const getTimeLeftToWorkMessage = (timeLeft) => {
    if (timeLeft === "0:00")
      return "You have worked too much already! Stop working this instant!";

    if (timeLeft.includes("NaN")) {
      return `If today is day one then see time left today below, otherwise enter times worked above.`;
    }

    return `You have ${timeLeft} this week.`;
  };

  const getTimeLeftToWorkTodayMessage = (timeLeft) => {
    if (timeLeft.includes("NaN")) {
      return `Please use the date time picker to select the time started today!`;
    }
    return `You have ${timeLeft} today.`;
  };

  const getTimeLeftToDailyMessage = (timeLeft) => {
    if (timeLeft.includes("NaN") || !timesWorkedThisWeekInput) return "";

    const daysWorked = timesWorkedThisWeekInput.split(",").length + 1;

    return `You have ${timeLeft} today to align with the ${daysWorked} day total.`;
  };

  const getMinutesTotalForWorkDays = (workDays) => {
    return Math.ceil(7.6 * workDays * 60);
  };

  // Inputs
  let timeStartedTodayInput = "";
  let timesWorkedThisWeekInput;
  let amountOfBreakInput = "0:00";
  let daysToWork = 0;

  // Global constants
  $: minutesInWorkWeek = getMinutesTotalForWorkDays(daysToWork);

  // Global variables
  let timeLeftToWorkMessage = "Click the button to update the times left";
  let timeLeftToWorkDailyMessage = "";
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
    if (!timeWorkedThisWeekInput) return minutesInWorkWeek;
    const timeWorked = timeWorkedThisWeekInput.split(",");

    const timeWorkedInMinutes = timeWorked.map((time) =>
      convertTimeToMinutes(time)
    );

    const totalTimeWorkedInMinutes = timeWorkedInMinutes.reduce(
      (a, b) => a + b
    );
    return minutesInWorkWeek - totalTimeWorkedInMinutes;
  };

  const getRemainingMinutesToWorkTodayBasedOnWeekSoFar = (
    timeWorkedThisWeekInput
  ) => {
    if (!timeWorkedThisWeekInput) return minutesInWorkWeek;
    const timeWorked = timeWorkedThisWeekInput.split(",");

    const timeWorkedInMinutes = timeWorked.map((time) =>
      convertTimeToMinutes(time)
    );

    const minutesInDaysWorked = getMinutesTotalForWorkDays(
      timeWorked.length + 1
    );

    const totalTimeWorkedInMinutes = timeWorkedInMinutes.reduce(
      (a, b) => a + b
    );
    return minutesInDaysWorked - totalTimeWorkedInMinutes;
  };

  let minutesRemainingInWorkWeek = 0;
  let minutesRemainingTodayBasedOnWeekSoFar = 0;
  let minutesRemainingToday = 0;

  const updateMessage = (
    timeStartedToday,
    timesWorkedThisWeek,
    amountOfBreak,
    timeNow,
    minutesInWorkWeek
  ) => {
    const minutesOfBreak = convertTimeToMinutes(amountOfBreak);

    minutesRemainingInWorkWeek =
      getRemainingMinutesToWorkInWeek(timesWorkedThisWeek, minutesInWorkWeek) +
      minutesOfBreak -
      timeBetweenStartedAndNowInMinutes;

    minutesRemainingToday =
      getMinutesRemainingToday(timeStartedToday) + minutesOfBreak;

    minutesRemainingTodayBasedOnWeekSoFar =
      getRemainingMinutesToWorkTodayBasedOnWeekSoFar(timesWorkedThisWeek) +
      minutesOfBreak -
      timeBetweenStartedAndNowInMinutes;

    const timeLeft = convertMinutesToTime(minutesRemainingInWorkWeek);
    const timeLeftToday = convertMinutesToTime(minutesRemainingToday);
    workedToday = convertMinutesToTime(
      timeBetweenStartedAndNowInMinutes - minutesOfBreak,
      true
    );
    const timeLeftTodayBasedOnWeekSoFar = convertMinutesToTime(
      minutesRemainingTodayBasedOnWeekSoFar
    );

    if (
      timesWorkedThisWeek &&
      timesWorkedThisWeek.split(",").length === daysToWork - 1
    ) {
      timeLeftToWorkMessage = "";
      timeLeftToWorkDailyMessage = "";
      timeLeftToWorkTodayMessage = getTimeLeftToWorkTodayMessage(timeLeft);
    } else {
      timeLeftToWorkMessage = getTimeLeftToWorkMessage(timeLeft);
      timeLeftToWorkTodayMessage = getTimeLeftToWorkTodayMessage(timeLeftToday);
      timeLeftToWorkDailyMessage = getTimeLeftToDailyMessage(
        timeLeftTodayBasedOnWeekSoFar
      );
    }
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

  const getFinishingTime = (
    timeNow,
    minutesRemainingToday,
    minutesRemainingTodayBasedOnWeekSoFar,
    minutesRemainingInWorkWeek
  ) => {
    let daysWorked = 0;
    if (timesWorkedThisWeekInput)
      daysWorked = timesWorkedThisWeekInput.split(",").length;

    if (daysWorked === daysToWork - 1)
      return new Date(timeNow.getTime() + minutesRemainingInWorkWeek * 60000);

    if (daysWorked === 0)
      return new Date(timeNow.getTime() + minutesRemainingToday * 60000);

    return new Date(
      timeNow.getTime() + minutesRemainingTodayBasedOnWeekSoFar * 60000
    );
  };

  $: finishTime = getFinishingTime(
    timeNow,
    minutesRemainingToday,
    minutesRemainingTodayBasedOnWeekSoFar,
    minutesRemainingInWorkWeek
  );
</script>

<main>
  <h1>Time left</h1>
  <h2>Time Now</h2>
  <h3>
    {timeNow.toLocaleTimeString()}
  </h3>
  <h2>
    {finishTime == "Invalid Date" ? "" : "Finish Time"}
  </h2>
  <h3>
    {finishTime == "Invalid Date"
      ? ""
      : finishTime.toLocaleString([], {
          day: "numeric",
          month: "numeric",
          year: "numeric",
          hour: "numeric",
          minute: "numeric",
        })}
  </h3>
  <div class="container">
    <p>How many days will you work this week?</p>
    <select bind:value={daysToWork}>
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
        {timeLeftToWorkDailyMessage}
      </strong>
    </p>
    <p>
      <strong>
        {timeLeftToWorkTodayMessage}
      </strong>
    </p>
    <p>
      <strong>
        {timeLeftToWorkMessage}
      </strong>
    </p>
    {#if !workedToday.includes("NaN")}
      <p>
        You've worked {workedToday} today.
      </p>
    {/if}
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
    margin-bottom: 50px;
  }
</style>
