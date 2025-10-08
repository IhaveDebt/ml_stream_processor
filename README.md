/**
 * Real-time Machine Learning Stream Processor
 *
 * Simulates ingesting data from multiple sensors,
 * processing them through a statistical anomaly detector,
 * and broadcasting alerts when thresholds are exceeded.
 */

type SensorReading = {
  id: string;
  timestamp: number;
  value: number;
};

class StreamProcessor {
  private windowSize: number;
  private readings: Map<string, number[]> = new Map();

  constructor(windowSize: number = 10) {
    this.windowSize = windowSize;
  }

  addReading(reading: SensorReading) {
    if (!this.readings.has(reading.id)) this.readings.set(reading.id, []);
    const arr = this.readings.get(reading.id)!;
    arr.push(reading.value);
    if (arr.length > this.windowSize) arr.shift();

    const mean = arr.reduce((a, b) => a + b, 0) / arr.length;
    const variance = arr.reduce((a, b) => a + Math.pow(b - mean, 2), 0) / arr.length;
    const stddev = Math.sqrt(variance);

    if (Math.abs(reading.value - mean) > 2.5 * stddev && arr.length >= 5) {
      console.warn(`[ALERT] Sensor ${reading.id} anomaly detected! Value=${reading.value.toFixed(2)}, Mean=${mean.toFixed(2)}`);
    }
  }
}

if (require.main === module) {
  const processor = new StreamProcessor(20);
  setInterval(() => {
    const r: SensorReading = {
      id: 'sensor-A',
      timestamp: Date.now(),
      value: Math.random() * 100 + (Math.random() > 0.9 ? 100 : 0)
    };
    processor.addReading(r);
  }, 200);
}
