// client/src/components/Timer.jsx
import React, { useState } from 'react';
import { DateTime } from 'luxon';

const Timer = () => {
  const [laps, setLaps] = useState([]);
  const [isRunning, setIsRunning] = useState(false);
  const [elapsed, setElapsed] = useState(0);
  const [selectedTimeZone, setSelectedTimeZone] = useState('UTC');

  // 타이머 시작
  const startTimer = () => {
    const startTime = Date.now();
    const interval = setInterval(() => {
      setElapsed(Date.now() - startTime);
    }, 10);
    setIsRunning(true);
  };

  // 랩 기록
  const recordLap = (title, content, tags) => {
    const now = DateTime.utc();
    const localTime = now.setZone(selectedTimeZone);
    const newLap = {
      id: crypto.randomUUID(),
      title,
      content,
      start_time: now.toISO(),
      end_time: now.plus({ ms: elapsed }).toISO(),
      local_start: localTime.toFormat("yyyy-MM-dd HH:mm:ss"),
      local_end: localTime.plus({ ms: elapsed }).toFormat("yyyy-MM-dd HH:mm:ss"),
      tags
    };
    setLaps([...laps, newLap]);
    // 백엔드로 전송
    fetch('/api/laps', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(newLap)
    });
  };

  return (
    <div>
      <button onClick={startTimer}>시작</button>
      <input type="text" placeholder="제목" />
      <input type="text" placeholder="내용" />
      <button onClick={() => recordLap("작업", "설명", ["태그1"])}>랩</button>
    </div>
  );
}

// client/src/components/Calendar.jsx
import React, { useEffect, useState } from 'react';
import FullCalendar from '@fullcalendar/react';
import dayGridPlugin from '@fullcalendar/daygrid';

const Calendar = () => {
  const [events, setEvents] = useState([]);

  useEffect(() => {
    // 백엔드에서 랩 데이터 가져오기
    fetch('/api/laps')
      .then(res => res.json())
      .then(data => {
        const formattedEvents = data.map(lap => ({
          title: lap.title,
          start: lap.start_time,
          end: lap.end_time
        }));
        setEvents(formattedEvents);
      });
  }, []);

  return (
    <FullCalendar
      plugins={[dayGridPlugin]}
      initialView="dayGridWeek"
      events={events}
    />
  );
};
