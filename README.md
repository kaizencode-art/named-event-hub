# named-event-hub

[![npm version](https://img.shields.io/npm/v/event-bridge.svg)](https://www.npmjs.com/package/event-bridge)
[![Apache-2.0 License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![TypeScript](https://img.shields.io/badge/%3C%2F%3E-TypeScript-%230074c1.svg)](https://www.typescriptlang.org/)

A type-safe event system enforcing structured communication patterns through single emitters and controlled consumers. Built on modern web standards with DOM compatibility.

## Features

- **Single Emitter Enforcement** - Prevent duplicate event sources
- **Consumer Limits** - Configurable listener counts
- **Collision-Free Namespacing** - `emitter:event` pattern
- **Lifecycle Tracking** - Explicit consumer relationships
- **Zod Validation** - Runtime type safety

## Installation

```bash
npm install event-bridge
```
## Usage
Basic Flow
```typescript

import { EventRegistry, EventManager } from 'event-bridge';

// 1. Register allowed events
window.EventRegistry.register('dataLoad');

// 2. Create named emitter
const loader = Object.assign(new EventTarget(), { name: 'DataLoader' });
EventManager.registerEmitter(loader, 'dataLoad');

// 3. Add consumer
const dashboard = Object.assign(new EventTarget(), { name: 'Dashboard' });

const handler = (e: CustomEvent) => {
  console.log('Received data:', e.detail);
};

EventManager.consume(dashboard, 'dataLoad', handler);

// 4. Dispatch event
EventManager.dispatch(loader, 'dataLoad', {
  detail: { items: [...] }
});

// 5. Cleanup
EventManager.removeConsume(dashboard, 'dataLoad', handler);
```

API Reference
EventRegistry
```typescript

interface EventRegistryAPI {
  /** Register new event type with Zod validation */
  register(eventName: string): void;
  
  /** Check event registration status */
  has(eventName: string): boolean;
  
  /** List all registered events */
  list(): readonly string[];
}
```
EventManager
```typescript

class EventManager {
  /** 
   * Register event emitter 
   * @param consumerLimit - Maximum allowed consumers (default: 1)
   */
  static registerEmitter(
    emitter: NamedEventTarget,
    eventName: string,
    consumerLimit?: number
  ): void;

  /** Dispatch namespaced event */
  static dispatch(
    emitter: NamedEventTarget,
    eventName: string,
    options: CustomEventInit
  ): void;

  /** Add event consumer */
  static consume(
    consumer: NamedEventTarget,
    eventName: string,
    callback: EventListener
  ): void;

  /** Remove consumer registration */
  static removeConsume(
    consumer: NamedEventTarget,
    eventName: string,
    callback: EventListener
  ): void;
}
```
Web Components Example
```typescript

import { LitElement } from 'lit';
import { EventManager } from 'event-bridge';

class DataSource extends LitElement implements NamedEventTarget {
  name = 'data-source';

  connectedCallback() {
    super.connectedCallback();
    EventManager.registerEmitter(this, 'dataReady');
  }

  private async loadData() {
    const data = await fetchData();
    EventManager.dispatch(this, 'dataReady', {
      detail: data,
      composed: true
    });
  }
}

class DataDisplay extends LitElement implements NamedEventTarget {
  name = 'data-display';

  connectedCallback() {
    super.connectedCallback();
    EventManager.consume(this, 'dataReady', this.handleData);
  }

  disconnectedCallback() {
    EventManager.removeConsume(this, 'dataReady', this.handleData);
    super.disconnectedCallback();
  }

  private handleData = (e: CustomEvent) => {
    this.renderData(e.detail);
  };
}
```
Error Handling
```typescript

// Handle registration errors
try {
  EventManager.registerEmitter(existingEmitter, 'dataUpdate');
} catch (error) {
  if (error instanceof DuplicateEmitterError) {
    console.warn('Emitter already registered:', error.message);
  }
}

// Handle invalid event names
try {
  window.EventRegistry.register('invalid:event');
} catch (error) {
  console.error('Validation failed:', error.message);
}
```
## Development
```bash

# Install dependencies
npm ci

# Build project
npm run build

# Run tests
npm test
```
## License

KaizenCode 2025

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this software except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
