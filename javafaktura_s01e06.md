%title: 
%author: Paweł 'teebor' Balczyński
%date: 2019-06-04


->  ┬┌─┐┬  ┬┌─┐┌─┐┌─┐┬┌─┌┬┐┬ ┬┬─┐┌─┐ <-
->  │├─┤└┐┌┘├─┤├┤ ├─┤├┴┐ │ │ │├┬┘├─┤ <-
-> └┘┴ ┴ └┘ ┴ ┴└  ┴ ┴┴ ┴ ┴ └─┘┴└─┴ ┴ <-

-> *s01e06: java.time ~~i java.nio~~* <-

**************************************************************

-> # agenda <-

- wstęp (o formie i zmianach w prezentacji)
- historia i motywacja zmian w podsystemie dat
- omwówienie java.time (z przykładami) 
- -java.io/nio (nio2)-

**************************************************************

-> # o mnie <-

- [linkedIn](https://www.linkedin.com/in/pbalczynski/)


**************************************************************

-> # historia, problemy i motywacje <-

- skomplikowana i nieoczywista w użyciu implementacja

- brak niezmienności (immutability) utrudniający wielowątkowość

- rozproszenie 
    - data: java.util.Date i java.sql.[Date, Time, Timestamp]
    - kalendarz: java.util.[Calendar, GregorianCalendar, TimeZone, SimpleTimeZone]
    - formatowanie: java.text.[DateFormat, SimpleDateFormat, DateFormatSymbols]

**************************************************************

-> # data przed javą 8 <-

```
new Date()

``````

```
new Date(2019,06,04)

``````

```
new Date(1557849600000L)

``````

**************************************************************

-> # data przed javą 8 c.d. <-

```
import java.text.SimpleDateFormat;
import java.util.concurrent.TimeUnit;
 
var dateFormat = new SimpleDateFormat("yyyy-MM-dd, HH:mm");
var lastJavafakturaOldWay = dateFormat.parse("2019-05-14, 18:00");
var todayJavafakturaOldWay = dateFormat.parse("2019-06-04, 18:00");
 
var diffInMilis = todayJavafakturaOldWay.getTime() - lastJavafakturaOldWay.getTime();
 
diffInMilis/1000/60/60/24
``````
<br>

```
TimeUnit.DAYS.convert(diffInMilis, TimeUnit.MILLISECONDS); [java 1.5]
``````
<br>

```
var cal = Calendar.getInstance();
cal.setTime(todayJavafakturaOldWay);
cal.add(Calendar.WEEK_OF_MONTH, -3);
cal.getTime();
``````

**************************************************************

-> # istniejące rozwiązania <-

- [commons.lang(3).time](https://commons.apache.org)
- [date4j](http://www.date4j.net), 
- [joda.time](https://www.joda.org/joda-time),
- [threeten.org](https://www.threeten.org)


**************************************************************


-> # Przegląd API JSR 310 <-

- kalendarz Gregoriański zgodny z ISO-8601
- wykorzystuje CLDR (Unicode Common Locale Data Repository) 
- wykorzystuje TZDB (Time-Zone Database)
- bezpieczne w aplikacjach wielowątkowych
- fluent API

**************************************************************

-> # Główne pakiety <-

- *java.time*
    główne klasy: np. LocalDate, LocalTime, LocalDateTime, Instant, Duration, Period

- *java.time.chrono*
    api dla kalendarzy innych niż ISO; Hijrah, Japanese, Minguo, Thai Buddhist

- *java.time.format*
    klasy do wyświetlania i parsowania dat: np. DateTimeFormatter, TextStyle

- *java.time.temporal*
    dodatkowe funkcjonalności do operowania czasem: np.Temporal, TemporalAdjuster, 
    ChronoField, ChronoUnit

- *java.time.zone*
    wsparcie dla stref czasowych

**************************************************************

-> # Data <-

- LocalDate
- YearMonth
- MonthDay
- Year
<br>
- DayOfWeek
- Month

**************************************************************

-> # Data c.d.<-

```
LocalDate.of(2019, Month.JUNE, 04);
 
YearMonth.of(2019, Month.JUNE);
 
MonthDay.of(Month.JUNE, 04);
 
Year.of(2019);
```
<br>

```
DayOfWeek.TUESDAY.plus(14)
 
Month.JULY.maxLength()
```

**************************************************************

-> # Data c.d.<-

```
LocalDate.now()
LocalDate.of(...)
LocalDate.parse(...)
LocalDate.from(TemporalAccessor temporal)
 
Year.isLeap(2020);
Year.of(2020).length();

```

**************************************************************

-> # Data i Czas <-

- LocalTime
- LocalTimeDate

**************************************************************

-> # Data i Czas c.d. <-

```
LocalDateTime.now()
LocalDateTime.ofInstant(Instant.now(), ZoneId.systemDefault()));

LocalDateTime.plus*(...)
LocalDateTime.minus*(...)
```

**************************************************************

-> # Strefa czasowa i przesunięcie <-

- ZoneId
- ZoneOffset
- ZonedDateTime
- OffsetDateTime
- OffsetTime


**************************************************************

-> # Strefa czasowa i przesunięcie <-

```
ZoneId.getAvailableZoneIds().stream()
    .filter(s -> s.contains("Europe"))
    .map(z -> ZoneId.of(z))
    .forEach(
        s -> System.out.println(zdt.withZoneSameInstant(s))
    )
```

**************************************************************

-> # Instatnt <-

> This class models a single instantaneous point on the time-line.
> This might be used to record event time-stamps in the application.

```
Instant.now()
 
Instant.ofEpochSecond(0L).until(Instant.now(), ChronoUnit.SECONDS);
 
Instant.from(...
Instant.parse(...
```
<br>

```
.plus*(...
.minus*(...
.isAfter(...
.isBefore(...
```

**************************************************************

-> # Parsowanie i formatowanie dat <-

- DateTimeFormatter
- TextStyle

```
var dtFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd, HH:mm");
LocalDateTime.parse("2019-06-04, 18:50", dtFormatter);
 
LocalDateTime.now().format(dtFormatter);
```
<br>

```
DayOfWeek dow = DayOfWeek.TUESDAY;
Locale locale = Locale.getDefault();
System.out.println(dow.getDisplayName(TextStyle.FULL, locale));
System.out.println(dow.getDisplayName(TextStyle.NARROW, locale));
System.out.println(dow.getDisplayName(TextStyle.SHORT, locale));
```

**************************************************************

-> # TemporalAdjuster(s) <-

```
@FunctionalInterface
public interface TemporalAdjuster {
    Temporal adjustInto(Temporal temporal);
}
```
<br>

```
now.with(TemporalAdjusters.firstDayOfMonth());
now.with(TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY));
now.with(TemporalAdjusters.lastDayOfMonth());
now.with(TemporalAdjusters.firstDayOfNextMonth()).getDayOfWeek();
now.with(TemporalAdjusters.firstDayOfYear()).getDayOfWeek();
now.with(TemporalAdjusters.firstDayOfNextYear());
```

**************************************************************

-> # TemporalAdjuster(s) <-

- Temporal with(TemporalAdjuster adjuster)
<br>

```
var nextBloodDonationAdjuster = TemporalAdjusters.ofDateAdjuster(in -> {
    var nextDonation = in.plusWeeks(8);
    if(nextDonation.getDayOfWeek() == DayOfWeek.SATURDAY || nextDonation.getDayOfWeek() == DayOfWeek.SUNDAY) {
        return nextDonation.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
    }
    return nextDonation;
});
 
LocalDate.now()
    .with(TemporalAdjusters.next(DayOfWeek.SATURDAY))
    .with(nextBloodDonationAdjuster)
```

<br>

```
Temporal nextFriday13(Temporal in) {
    var friday = LocalDate.from(in).with(TemporalAdjusters.next(DayOfWeek.FRIDAY));
    if(friday.getDayOfMonth() == 13)
        return friday;
    else
        return nextFriday13(friday);
}
 
LocalDate.now().with(d -> nextFriday13(d))
```

**************************************************************

-> # TemporalQuery <-

```
var query = TemporalQueries.precision();
LocalDate.now().query(query);
LocalDateTime.now().query(query);
Year.now().query(query);
YearMonth.now().query(query);
Instant.now().query(query);
```

**************************************************************

-> # Duration, Period <-

- Duration - przedział czasowy mierzony w jednostkach bazujących na czasie (sekundy, minuty itp.)
- Period - przedział czasowy mierzony w jednostkach bazujących na dacie (dni, miesiące itp.)
- ChronoUnit.XXX.between - w przypadku kiedy interesuje nas tylko jedna jednostka

```
var nastepnaJavafaktura = LocalDateTime.of(2019,06,04,18,00).plusWeeks(1);
 
Duration.between(LocalDateTime.now(), nastepnaJavafaktura)
```

```
var java1 = LocalDate.of(1996,1,23);
var joda09 = LocalDate.of(2003,12,16);
var jsr310 = LocalDate.of(2014,3,4);
var java8 = LocalDate.of(2014,3,18);

Period.between(joda09, java1);
Period.between(joda09,jsr310).toTotalMonths();
Period.between(jsr310, java8).getDays();
Period.between(jsr310, java8);

ChronoUnit.YEARS.between(java1,java8);
```

**************************************************************

-> # Kompatabilność ze starymi klasami <-

- Calendar.toInstant() konwertuje Calendar na Instant
- GregorianCalendar.toZonedDateTime() konwertuje GregorianCalendar na ZonedDateTime
- GregorianCalendar.from(ZonedDateTime) tworzy GregorianCalendar (z domyślnymi lokalami) z ZonedDateTime
- Date.from(Instant) tworzy Date z Instant-a
- Date.toInstant() konwertuje Date na Instant
- TimeZone.toZoneId() konwertuje TimeZone na ZoneId

**************************************************************

-> # Co ominęliśmy <-

- Clock
- java.time.chrono i konwersje pomiędzy datami innymi niż ISO
  

**************************************************************

-> # Warto zapamiętać <-

- dobra dokumentacja Oracle zawierająca tutoriale i przykłady
- wielowątkowo-bezpieczna implementacja 
- dostępna wersja dla javy 6 i 7 (threetenbp)


**************************************************************

# przydatne linki 1 

- [tutorial/datetime](https://docs.oracle.com/javase/tutorial/datetime)
- [java-8-date-time-api-tutorial](https://examples.javacodegeeks.com/core-java/time/java-8-date-time-api-tutorial)

**************************************************************

-> # przydatne linki 2 [IO/NIO/NIO2]

- [technotes/guides](https://docs.oracle.com/javase/8/docs/technotes/guides/io/index.html)
- [j-nio](https://developer.ibm.com/tutorials/j-nio/)
- [j-nio2-1](https://www.ibm.com/developerworks/java/library/j-nio2-1/)
- [tutorials.jenkov.com/java-nio](http://tutorials.jenkov.com/java-nio/index.html)
- [core-java-master-merlin-s-new-i-o-classes](https://www.javaworld.com/article/2075575/core-java-master-merlin-s-new-i-o-classes.html)
- [Buffers !](https://www.slideshare.net/BalamuruganSoundararajan/nio-and-nio2)

**************************************************************

-> # informacje dodatkowe <-

- [JSR 310: Date and Time API](https://jcp.org/en/jsr/detail?id=310)
- [Date and time format - ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html)
- [java-simpledateformat-thread-safety-issues](https://www.callicoder.com/java-simpledateformat-thread-safety-issues/)
- [ThreeTen threetenbp](https://www.threeten.org/threetenbp)

- [Java-JDK-1.0-src](https://github.com/barismeral/Java-JDK-1.0-src)

**************************************************************

-> # Dzięki <-

**************************************************************
