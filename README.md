// hooks/useSimulation.js
import { useState, useEffect, useRef, useMemo } from 'react';

export const useSimulation = (allTripsData) => {
  const [simTime, setSimTime] = useState(0);
  const [isPlaying, setIsPlaying] = useState(false);
  const [speed, setSpeed] = useState(1); // 1x, 5x, 10x
  
  // Initialize simTime to the earliest event across all trips
  useEffect(() => {
    const startTimes = allTripsData.flatMap(trip => trip[0]?.timestamp || []);
    setSimTime(Math.min(...startTimes));
  }, [allTripsData]);

  useEffect(() => {
    let interval;
    if (isPlaying) {
      interval = setInterval(() => {
        setSimTime(prev => prev + (1000 * speed)); // Advance clock
      }, 1000); // Update every real second
    }
    return () => clearInterval(interval);
  }, [isPlaying, speed]);

  // Derive current state of the fleet
  const fleetState = useMemo(() => {
    return allTripsData.map(events => {
      const pastEvents = events.filter(e => e.timestamp <= simTime);
      const latest = pastEvents[pastEvents.length - 1];
      const completion = (pastEvents.length / events.length) * 100;
      
      return {
        id: events[0]?.tripId,
        current: latest,
        history: pastEvents,
        progress: completion,
        status: latest?.status || 'UNKNOWN'
      };
    });
  }, [simTime, allTripsData]);

  return { simTime, fleetState, isPlaying, setIsPlaying, setSpeed, speed };
};
