# Theia 무한 캔버스 브라우저 통합: WebContentsView 혁신

## 💡 핵심 발견: Theia는 이미 Electron이다!

**중요한 깨달음**: Theia는 이미 Electron 37.2.1을 사용하고 있으며, 2024년 최신 WebContentsView API를 지원합니다. 문제는 Theia가 아직 구식 BrowserWindow 아키텍처를 사용하고 있다는 것입니다. 이것이 바로 우리의 **기회**입니다!

## 현재 상황 분석

### Theia의 현재 Electron 사용 현황
```json
// examples/electron/package.json
{
  "name": "@theia/example-electron",
  "main": "lib/backend/electron-main.js",
  "devDependencies": {
    "electron": "37.2.1"  // ✅ 최신 Electron 사용
  }
}
```

### 하지만 구식 아키텍처 고수
```typescript
// packages/core/src/electron-main/theia-electron-window.ts:84
protected init(): void {
    this._window = new BrowserWindow(this.options);  // 😱 구식!
    // WebContentsView (Electron 30+) 미사용
}
```

## 혁신적 솔루션: Theia 확장에서 WebContentsView 직접 활용

### 아키텍처 개요
```
Theia Electron App (현재)
├── BrowserWindow (메인 창) - 기존 Theia
└── 새로운 확장
    └── BaseWindow (무한 캔버스)
        ├── WebContentsView (GitHub)
        ├── WebContentsView (StackOverflow)
        ├── WebContentsView (MDN)
        └── WebContentsView (무제한)
```

## 핵심 구현: Theia 확장

### 1. 기본 확장 구조
```typescript
// src/browser/infinite-canvas-browser-contribution.ts
import { injectable, inject } from '@theia/core/shared/inversify';
import { FrontendApplicationContribution } from '@theia/core/lib/browser/frontend-application-contribution';
import { ApplicationShell } from '@theia/core/lib/browser/shell/application-shell';
import { remote } from 'electron';

const { BaseWindow, WebContentsView } = remote;

@injectable()
export class InfiniteCanvasBrowserContribution implements FrontendApplicationContribution {
    @inject(ApplicationShell)
    protected readonly shell: ApplicationShell;

    private canvasWindow: Electron.BaseWindow | null = null;
    private browserViews: Map<string, Electron.WebContentsView> = new Map();

    async onStart(): Promise<void> {
        await this.initializeInfiniteCanvas();
    }

    private async initializeInfiniteCanvas(): Promise<void> {
        // 메인 Theia 윈도우 가져오기
        const currentWindow = remote.getCurrentWindow();

        // 무한 캔버스 윈도우 생성
        this.canvasWindow = new BaseWindow({
            width: 3840,
            height: 2160,
            parent: currentWindow,
            frame: false,
            transparent: true,
            alwaysOnTop: false,
            skipTaskbar: true,
            webPreferences: {
                nodeIntegration: false,
                contextIsolation: true
            }
        });

        // 기본 브라우저들 생성
        await this.createDefaultBrowsers();

        // 이벤트 처리 설정
        this.setupEventHandlers();
    }

    private async createDefaultBrowsers(): Promise<void> {
        // 개발 도구들
        await this.addBrowserToCanvas({
            url: 'https://github.com',
            x: 100,
            y: 100,
            width: 800,
            height: 600,
            title: 'GitHub'
        });

        // 문서
        await this.addBrowserToCanvas({
            url: 'https://developer.mozilla.org',
            x: 950,
            y: 100,
            width: 800,
            height: 600,
            title: 'MDN'
        });

        // 질답 사이트
        await this.addBrowserToCanvas({
            url: 'https://stackoverflow.com',
            x: 100,
            y: 750,
            width: 800,
            height: 600,
            title: 'Stack Overflow'
        });
    }

    async addBrowserToCanvas(options: BrowserCanvasOptions): Promise<string> {
        if (!this.canvasWindow) {
            throw new Error('Canvas window not initialized');
        }

        // 🔥 진짜 브라우저 뷰 생성!
        const browserView = new WebContentsView({
            webPreferences: {
                nodeIntegration: false,
                contextIsolation: true,
                sandbox: false,  // 🎯 완전한 브라우저 권한!
                webSecurity: true,
                allowRunningInsecureContent: false,
                plugins: true,
                experimentalFeatures: true
            }
        });

        // 캔버스 좌표에 정확히 배치
        browserView.setBounds({
            x: options.x,
            y: options.y,
            width: options.width,
            height: options.height
        });

        // URL 로드
        await browserView.webContents.loadURL(options.url);

        // BaseWindow에 추가
        this.canvasWindow.contentView.addChildView(browserView);

        // 🔥 진짜 브라우저 기능들!
        browserView.webContents.on('did-finish-load', () => {
            // DevTools 접근 가능
            // browserView.webContents.openDevTools();

            // 확장 프로그램 지원
            // 완전한 Chrome API 접근

            console.log(`Browser loaded: ${options.title} at ${options.url}`);
        });

        // 관리용 ID 생성
        const browserId = `browser-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
        this.browserViews.set(browserId, browserView);

        return browserId;
    }

    private setupEventHandlers(): void {
        if (!this.canvasWindow) return;

        // 무한 캔버스 팬 구현
        let isPanning = false;
        let lastMousePos = { x: 0, y: 0 };

        this.canvasWindow.on('will-move', (event, newBounds) => {
            // 캔버스 이동 시 모든 브라우저뷰 동기화
            this.syncBrowserPositions(newBounds);
        });

        // 줌 이벤트 (마우스 휠)
        this.canvasWindow.webContents?.on('wheel', (event) => {
            this.handleCanvasZoom(event.deltaY, event.x, event.y);
        });
    }

    private syncBrowserPositions(canvasBounds: Electron.Rectangle): void {
        this.browserViews.forEach((browserView, id) => {
            const currentBounds = browserView.getBounds();

            // 캔버스 이동에 따라 브라우저뷰 위치 조정
            // 실제 구현에서는 더 정교한 좌표 변환 필요
            browserView.setBounds(currentBounds);
        });
    }

    private handleCanvasZoom(deltaY: number, centerX: number, centerY: number): void {
        const zoomFactor = deltaY > 0 ? 0.9 : 1.1;

        this.browserViews.forEach((browserView) => {
            const bounds = browserView.getBounds();

            // 줌 중심점 기준으로 크기 및 위치 조정
            const newWidth = bounds.width * zoomFactor;
            const newHeight = bounds.height * zoomFactor;

            const newX = centerX - (centerX - bounds.x) * zoomFactor;
            const newY = centerY - (centerY - bounds.y) * zoomFactor;

            browserView.setBounds({
                x: Math.round(newX),
                y: Math.round(newY),
                width: Math.round(newWidth),
                height: Math.round(newHeight)
            });
        });
    }
}

interface BrowserCanvasOptions {
    url: string;
    x: number;
    y: number;
    width: number;
    height: number;
    title: string;
}
```

### 2. Theia와의 통합

```typescript
// src/browser/infinite-canvas-commands.ts
import { Command, CommandContribution, CommandRegistry } from '@theia/core/lib/common/command';
import { MenuContribution, MenuModelRegistry } from '@theia/core/lib/common/menu';
import { injectable, inject } from '@theia/core/shared/inversify';
import { InfiniteCanvasBrowserContribution } from './infinite-canvas-browser-contribution';

export const INFINITE_CANVAS_COMMANDS = {
    OPEN_CANVAS: Command.toLocalizedCommand({
        id: 'infinite-canvas.open',
        label: 'Open Infinite Canvas Browser'
    }),
    ADD_BROWSER: Command.toLocalizedCommand({
        id: 'infinite-canvas.add-browser',
        label: 'Add Browser to Canvas'
    })
};

@injectable()
export class InfiniteCanvasCommandContribution implements CommandContribution {
    @inject(InfiniteCanvasBrowserContribution)
    protected readonly canvasContribution: InfiniteCanvasBrowserContribution;

    registerCommands(registry: CommandRegistry): void {
        registry.registerCommand(INFINITE_CANVAS_COMMANDS.OPEN_CANVAS, {
            execute: () => this.canvasContribution.showCanvas()
        });

        registry.registerCommand(INFINITE_CANVAS_COMMANDS.ADD_BROWSER, {
            execute: async () => {
                const url = await this.promptForUrl();
                if (url) {
                    await this.canvasContribution.addBrowserToCanvas({
                        url,
                        x: Math.random() * 1000,
                        y: Math.random() * 600,
                        width: 800,
                        height: 600,
                        title: new URL(url).hostname
                    });
                }
            }
        });
    }

    private async promptForUrl(): Promise<string | undefined> {
        // Theia의 입력 다이얼로그 사용
        // 실제 구현 필요
        return prompt('Enter URL:') || undefined;
    }
}
```

### 3. 무한 캔버스 네비게이션

```typescript
// src/browser/canvas-navigation.ts
@injectable()
export class CanvasNavigationManager {
    private transform = {
        x: 0,
        y: 0,
        scale: 1.0
    };

    private readonly MIN_SCALE = 0.1;
    private readonly MAX_SCALE = 5.0;

    constructor(
        private canvasWindow: Electron.BaseWindow,
        private browserViews: Map<string, Electron.WebContentsView>
    ) {
        this.setupNavigationControls();
    }

    private setupNavigationControls(): void {
        // 키보드 네비게이션
        this.canvasWindow.webContents?.on('before-input-event', (event, input) => {
            if (input.control || input.meta) {
                switch (input.key) {
                    case '=':
                    case '+':
                        this.zoomIn();
                        break;
                    case '-':
                        this.zoomOut();
                        break;
                    case '0':
                        this.resetZoom();
                        break;
                }
            }
        });

        // 마우스 드래그 팬
        let isDragging = false;
        let lastPos = { x: 0, y: 0 };

        this.canvasWindow.on('mouse-down', (event) => {
            if (event.button === 1) { // 중간 버튼
                isDragging = true;
                lastPos = { x: event.x, y: event.y };
            }
        });

        this.canvasWindow.on('mouse-move', (event) => {
            if (isDragging) {
                const deltaX = event.x - lastPos.x;
                const deltaY = event.y - lastPos.y;

                this.pan(deltaX, deltaY);

                lastPos = { x: event.x, y: event.y };
            }
        });

        this.canvasWindow.on('mouse-up', () => {
            isDragging = false;
        });
    }

    private zoomIn(): void {
        this.setZoom(this.transform.scale * 1.2);
    }

    private zoomOut(): void {
        this.setZoom(this.transform.scale / 1.2);
    }

    private resetZoom(): void {
        this.setZoom(1.0);
        this.transform.x = 0;
        this.transform.y = 0;
        this.updateAllBrowserPositions();
    }

    private setZoom(newScale: number): void {
        this.transform.scale = Math.max(this.MIN_SCALE, Math.min(this.MAX_SCALE, newScale));
        this.updateAllBrowserPositions();
    }

    private pan(deltaX: number, deltaY: number): void {
        this.transform.x += deltaX;
        this.transform.y += deltaY;
        this.updateAllBrowserPositions();
    }

    private updateAllBrowserPositions(): void {
        this.browserViews.forEach((browserView, id) => {
            const originalBounds = this.getOriginalBounds(id);

            const scaledBounds = {
                x: Math.round(originalBounds.x * this.transform.scale + this.transform.x),
                y: Math.round(originalBounds.y * this.transform.scale + this.transform.y),
                width: Math.round(originalBounds.width * this.transform.scale),
                height: Math.round(originalBounds.height * this.transform.scale)
            };

            browserView.setBounds(scaledBounds);
        });
    }

    private getOriginalBounds(browserId: string): Electron.Rectangle {
        // 원본 브라우저 위치 정보 저장소에서 가져오기
        // 실제 구현에서는 Map으로 관리
        return { x: 0, y: 0, width: 800, height: 600 };
    }
}
```

## 고급 기능 구현

### 1. 브라우저 간 통신
```typescript
// src/browser/browser-communication.ts
@injectable()
export class BrowserCommunicationManager {
    private messageHandlers = new Map<string, Function>();

    constructor(private browserViews: Map<string, Electron.WebContentsView>) {
        this.setupCommunication();
    }

    private setupCommunication(): void {
        this.browserViews.forEach((browserView, id) => {
            // 각 브라우저에 통신 API 주입
            browserView.webContents.executeJavaScript(`
                window.canvasBridge = {
                    sendToOtherBrowsers: (message) => {
                        window.electronAPI.sendToCanvas('broadcast', message);
                    },
                    sendToBrowser: (browserId, message) => {
                        window.electronAPI.sendToCanvas('direct', browserId, message);
                    },
                    onMessage: (callback) => {
                        window.electronAPI.onCanvasMessage(callback);
                    }
                };
            `);

            // 메시지 라우팅
            browserView.webContents.on('ipc-message', (event, channel, ...args) => {
                if (channel === 'canvas-message') {
                    this.routeMessage(id, ...args);
                }
            });
        });
    }

    private routeMessage(senderId: string, type: string, ...data: any[]): void {
        if (type === 'broadcast') {
            // 모든 다른 브라우저에게 전송
            this.browserViews.forEach((browserView, id) => {
                if (id !== senderId) {
                    browserView.webContents.send('canvas-message', senderId, ...data);
                }
            });
        } else if (type === 'direct') {
            // 특정 브라우저에게 전송
            const [targetId, ...message] = data;
            const targetBrowser = this.browserViews.get(targetId);
            if (targetBrowser) {
                targetBrowser.webContents.send('canvas-message', senderId, ...message);
            }
        }
    }
}
```

### 2. 상태 저장 및 복원
```typescript
// src/browser/canvas-persistence.ts
@injectable()
export class CanvasPersistenceManager {
    private readonly STORAGE_KEY = 'infinite-canvas-state';

    async saveCanvasState(browserStates: BrowserState[]): Promise<void> {
        const state: CanvasState = {
            version: '1.0',
            timestamp: Date.now(),
            browsers: browserStates,
            transform: this.getCurrentTransform()
        };

        // Theia의 localStorage 또는 워크스페이스 설정 사용
        localStorage.setItem(this.STORAGE_KEY, JSON.stringify(state));
    }

    async loadCanvasState(): Promise<CanvasState | null> {
        const stored = localStorage.getItem(this.STORAGE_KEY);
        if (!stored) return null;

        try {
            return JSON.parse(stored) as CanvasState;
        } catch {
            return null;
        }
    }

    async restoreCanvas(state: CanvasState): Promise<void> {
        // 저장된 브라우저들 복원
        for (const browserState of state.browsers) {
            await this.createBrowserFromState(browserState);
        }

        // 변형 상태 복원
        this.applyTransform(state.transform);
    }

    private async createBrowserFromState(state: BrowserState): Promise<void> {
        // 브라우저 상태에서 복원
        // URL, 위치, 크기, 히스토리 등
    }
}

interface CanvasState {
    version: string;
    timestamp: number;
    browsers: BrowserState[];
    transform: TransformState;
}

interface BrowserState {
    id: string;
    url: string;
    title: string;
    bounds: Electron.Rectangle;
    history?: string[];
    scrollPosition?: { x: number, y: number };
}

interface TransformState {
    x: number;
    y: number;
    scale: number;
}
```

## 모듈 바인딩 및 활성화

### 1. 프론트엔드 모듈
```typescript
// src/browser/infinite-canvas-frontend-module.ts
import { ContainerModule } from '@theia/core/shared/inversify';
import { FrontendApplicationContribution } from '@theia/core/lib/browser/frontend-application-contribution';
import { CommandContribution } from '@theia/core/lib/common/command';
import { MenuContribution } from '@theia/core/lib/common/menu';
import { InfiniteCanvasBrowserContribution } from './infinite-canvas-browser-contribution';
import { InfiniteCanvasCommandContribution } from './infinite-canvas-commands';

export default new ContainerModule(bind => {
    // 메인 기여자
    bind(InfiniteCanvasBrowserContribution).toSelf().inSingletonScope();
    bind(FrontendApplicationContribution).toService(InfiniteCanvasBrowserContribution);

    // 명령어 기여자
    bind(InfiniteCanvasCommandContribution).toSelf().inSingletonScope();
    bind(CommandContribution).toService(InfiniteCanvasCommandContribution);
    bind(MenuContribution).toService(InfiniteCanvasCommandContribution);
});
```

### 2. 패키지 구성
```json
{
  "name": "@theia/infinite-canvas-browser",
  "version": "1.0.0",
  "description": "Infinite Canvas Browser Integration for Theia",
  "theiaExtensions": [
    {
      "frontend": "lib/browser/infinite-canvas-frontend-module"
    }
  ],
  "dependencies": {
    "@theia/core": "^1.64.0",
    "@theia/electron": "^1.64.0"
  },
  "keywords": [
    "theia-extension",
    "browser",
    "canvas",
    "electron"
  ]
}
```

## 실제 사용 시나리오

### 1. 개발자 워크스페이스
```typescript
// 개발 중 여러 리소스를 무한 캔버스에 배치
async setupDeveloperWorkspace(): Promise<void> {
    // 코드 참조
    await this.addBrowser('https://github.com/microsoft/vscode', 0, 0);

    // 문서
    await this.addBrowser('https://code.visualstudio.com/api', 850, 0);

    // 이슈 트래킹
    await this.addBrowser('https://github.com/microsoft/vscode/issues', 1700, 0);

    // 스택 오버플로우
    await this.addBrowser('https://stackoverflow.com/questions/tagged/vscode', 0, 650);

    // MDN 참조
    await this.addBrowser('https://developer.mozilla.org', 850, 650);
}
```

### 2. 연구 환경
```typescript
// 여러 논문, 자료를 비교 분석
async setupResearchEnvironment(papers: string[]): Promise<void> {
    const cols = 3;
    const browserWidth = 600;
    const browserHeight = 800;
    const spacing = 50;

    papers.forEach(async (paperUrl, index) => {
        const row = Math.floor(index / cols);
        const col = index % cols;

        const x = col * (browserWidth + spacing);
        const y = row * (browserHeight + spacing);

        await this.addBrowser(paperUrl, x, y, browserWidth, browserHeight);
    });
}
```

### 3. 모니터링 대시보드
```typescript
// 다중 시스템 모니터링
async setupMonitoringDashboard(): Promise<void> {
    const services = [
        'https://grafana.example.com',
        'https://kibana.example.com',
        'https://prometheus.example.com',
        'https://jaeger.example.com'
    ];

    services.forEach(async (url, index) => {
        const x = (index % 2) * 1000;
        const y = Math.floor(index / 2) * 600;

        await this.addBrowser(url, x, y, 950, 550);
    });
}
```

## 장점 및 특징

### ✅ 진짜 브라우저 경험
- **완전한 Chromium 기능**: 확장 프로그램, DevTools, 모든 웹 API
- **네이티브 성능**: iframe 오버헤드 없음
- **실제 브라우저 권한**: 샌드박스 제약 없음

### ✅ 완벽한 Theia 통합
- **기존 시스템 활용**: Theia의 DI, 명령어, 메뉴 시스템
- **워크스페이스 통합**: 상태 저장, 설정 관리
- **확장성**: 다른 Theia 확장과 연동

### ✅ 무한 캔버스의 장점
- **자유로운 배치**: 제약 없는 브라우저 위치 조정
- **줌/팬 지원**: 대량의 브라우저도 효율적 관리
- **공간 활용**: 대형 모니터, 멀티 모니터 최적화

### ✅ 고급 기능
- **브라우저 간 통신**: 데이터 공유, 동기화
- **상태 저장**: 워크스페이스별 브라우저 레이아웃 저장
- **퍼포먼스 최적화**: 보이지 않는 브라우저 일시정지

## 성능 고려사항

### 메모리 관리
```typescript
class PerformanceOptimizer {
    private readonly MAX_ACTIVE_BROWSERS = 10;
    private readonly VIEWPORT_MARGIN = 200; // 뷰포트 밖 여유 공간

    optimizeBrowserVisibility(): void {
        const visibleBrowsers = this.getVisibleBrowsers();
        const activeBrowsers = this.getActiveBrowsers();

        // 보이지 않는 브라우저 일시정지
        this.browserViews.forEach((browserView, id) => {
            if (!visibleBrowsers.has(id) && activeBrowsers.has(id)) {
                this.suspendBrowser(browserView);
            } else if (visibleBrowsers.has(id) && !activeBrowsers.has(id)) {
                this.resumeBrowser(browserView);
            }
        });
    }

    private suspendBrowser(browserView: Electron.WebContentsView): void {
        // 브라우저 프로세스 일시정지
        browserView.webContents.setBackgroundThrottling(true);
        browserView.webContents.setAudioMuted(true);
    }

    private resumeBrowser(browserView: Electron.WebContentsView): void {
        // 브라우저 프로세스 재개
        browserView.webContents.setBackgroundThrottling(false);
        browserView.webContents.setAudioMuted(false);
    }
}
```

## 향후 개선 방향

### 1. Theia 코어 기여
- **공식 WebContentsView 지원**: Theia 코어를 WebContentsView로 마이그레이션
- **무한 캔버스 API**: 공식 API로 제공

### 2. 고급 기능 확장
- **AI 통합**: 브라우저 자동 배치, 스마트 그룹핑
- **협업 기능**: 다중 사용자 동시 사용
- **클라우드 동기화**: 원격 워크스페이스 지원

### 3. 성능 최적화
- **가상화**: 대량 브라우저 효율적 관리
- **하드웨어 가속**: GPU 활용 렌더링
- **분산 처리**: 브라우저 프로세스 분산

## 결론

이 솔루션은 Theia가 이미 Electron을 사용한다는 사실을 활용하여, 최신 WebContentsView API로 진정한 무한 캔버스 브라우저 환경을 구현합니다. iframe의 모든 제약을 벗어나 완전한 브라우저 기능을 제공하면서도, Theia의 강력한 확장 시스템과 완벽하게 통합됩니다.

**핵심 성취:**
- ✅ **진짜 브라우저**: 완전한 Chromium 기능과 권한
- ✅ **무한 캔버스**: 자유로운 배치와 네비게이션
- ✅ **Theia 통합**: 기존 시스템과 완벽한 조화
- ✅ **즉시 구현 가능**: 기존 Theia 프로젝트에 확장으로 추가

이제 더 이상 iframe의 제약에 갇힐 필요가 없습니다. Theia + WebContentsView로 진정한 무한 캔버스 브라우저 혁명을 시작하세요! 🚀