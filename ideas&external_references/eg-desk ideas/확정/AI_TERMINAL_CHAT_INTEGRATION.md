# AI Terminal Chat Integration Ideas

## 개요
Claude Code / Gemini CLI를 Theia 앱 내에서 터미널 스타일 UI로 통합하는 방법

## 핵심 패턴 (Theia AI Terminal 분석 결과)

### 1. Overlay 패턴 (현재 AI Terminal 방식)
**위치**: `packages/ai-terminal/src/browser/ai-terminal-contribution.ts:87-209`

```typescript
class AiTerminalChatWidget {
    constructor(terminalWidget, terminalAgent) {
        // 터미널 위에 HTML 오버레이 생성
        this.chatContainer = document.createElement('div');
        terminalWidget.node.appendChild(this.chatContainer);

        // 입력창 + 결과창
        // Ctrl+I로 활성화
        // AI Agent 호출 → 터미널에 명령어 전송
    }
}
```

**장점**: 기존 터미널 그대로 유지
**단점**: 오버레이가 터미널 위를 가림

---

## 제안 방법들

### 방법 1: xterm.js 기반 채팅 터미널 (추천)

터미널처럼 보이지만 실제로는 AI 채팅 UI

```typescript
@injectable()
export class AIChatTerminalWidget extends TerminalWidget {
    constructor(
        @inject(ClaudeCodeWrapper) claudeWrapper,
        @inject(GeminiCLIWrapper) geminiWrapper,
        @inject(CanvasContextService) canvasContext
    ) {
        super();
        this.initChatMode();
    }

    protected initChatMode() {
        // xterm.js 터미널이지만 입력을 가로채서 AI와 통신
        this.terminal.onData(data => {
            if (data === '\r') { // Enter
                this.handleAIChat(this.currentInput);
            } else {
                this.currentInput += data;
                this.terminal.write(data);
            }
        });

        this.write('AI Chat Terminal\r\n> ');
    }

    async handleAIChat(input: string) {
        // 1. 사용자 메시지 표시 (색상 구분)
        this.writeLine(`\x1b[32m➤ You:\x1b[0m ${input}`);

        // 2. 앱 컨텍스트 수집
        const context = {
            canvas: this.canvasContext.getCurrentState(),
            selectedElements: this.canvasContext.getSelected(),
            cwd: await this.cwd
        };

        // 3. Claude Code / Gemini CLI 호출
        const response = await this.claudeWrapper.sendWithContext(input, context);

        // 4. AI 응답 표시
        this.writeLine(`\x1b[36m◆ AI:\x1b[0m ${response}\r\n`);

        // 5. 새 프롬프트
        this.write('> ');
        this.currentInput = '';
    }
}
```

**UI 예시**:
```
AI Chat Terminal
> how do I create a new node?
➤ You: how do I create a new node?
◆ AI: You can create a new node by clicking on the canvas...

>
```

---

### 방법 2: Split View (터미널 + 채팅)

실제 터미널과 AI 채팅을 분할 화면으로

```typescript
@injectable()
export class HybridTerminalChatWidget extends SplitPanel {
    constructor(
        @inject(TerminalWidget) realTerminal,
        @inject(AIChatTerminalWidget) chatTerminal
    ) {
        super({ orientation: 'vertical' });

        // 상단: 실제 터미널 (명령 실행)
        this.addWidget(realTerminal);

        // 하단: AI 채팅 (터미널 스타일)
        this.addWidget(chatTerminal);

        // 비율 조정 가능
        this.setRelativeSizes([0.6, 0.4]);
    }
}
```

**UI 레이아웃**:
```
┌─────────────────────────────────────┐
│  Terminal (실제 bash/zsh)            │
│  $ npm install                      │
│  $ git status                       │
├─────────────────────────────────────┤
│  AI Chat (터미널 스타일)              │
│  > how do I create a circle?        │
│  ◆ AI: Use canvas.createCircle()   │
│  >                                  │
└─────────────────────────────────────┘
```

---

### 방법 3: 통합 터미널 (고급)

하나의 터미널에서 모드 전환

```typescript
class UnifiedTerminalWidget extends TerminalWidget {
    mode: 'shell' | 'ai-chat' = 'shell';

    protected onData(data: string) {
        if (data.startsWith('.claude') || data.startsWith('.gemini')) {
            this.switchToAIMode();
        } else if (this.mode === 'ai-chat' && data === 'exit') {
            this.switchToShellMode();
        } else if (this.mode === 'ai-chat') {
            this.handleAIChat(data);
        } else {
            // 일반 터미널 동작
            super.onData(data);
        }
    }
}
```

**사용 예시**:
```
$ ls
file1.txt  file2.txt

$ .claude
Entering AI Chat mode (type 'exit' to return)
> how do I find all TypeScript files?
◆ AI: Use: find . -name "*.ts"
> exit

$ find . -name "*.ts"
...
```

---

## 핵심 컴포넌트

### 1. Terminal Widget 기반
- **xterm.js**: 터미널 렌더링
- **node-pty**: 실제 shell 프로세스 (필요시)
- **TerminalWidget**: Theia 추상 레이어

### 2. AI Wrapper
```typescript
@injectable()
export class ClaudeCodeWrapper {
    private ptyProcess: IPty;

    async sendWithContext(prompt: string, context: any): Promise<string> {
        // 1. node-pty로 claude-code 프로세스 시작
        // 2. context를 CLAUDE.md 형태로 전달
        // 3. 응답 스트리밍
        // 4. 파싱 후 반환
    }
}
```

### 3. Context Provider
```typescript
@injectable()
export class AppContextService {
    collectContext(): AppContext {
        return {
            canvas: this.canvasService.getState(),
            selectedElements: this.selectionService.getSelected(),
            recentActions: this.historyService.getRecent(10),
            openFiles: this.editorService.getOpenEditors()
        };
    }
}
```

---

## 참고 코드 위치

### Theia AI Terminal
- Agent 구현: `packages/ai-terminal/src/browser/ai-terminal-agent.ts`
- UI 오버레이: `packages/ai-terminal/src/browser/ai-terminal-contribution.ts:87-209`
- 터미널 버퍼 읽기: `contribution.ts:187-192`

### Theia AI Chat UI
- 채팅 위젯: `packages/ai-chat-ui/src/browser/chat-view-widget.tsx`
- 입력 위젯: `packages/ai-chat-ui/src/browser/chat-input-widget.tsx`
- 메시지 렌더러: `packages/ai-chat-ui/src/browser/chat-response-renderer/`

### Theia Terminal
- 터미널 위젯: `packages/terminal/src/browser/base/terminal-widget.ts`
- 구현체: `packages/terminal/src/browser/terminal-widget-impl.ts`
- API: `sendText()`, `buffer.getLines()`, `onOutput`, `onData`

---

## 구현 우선순위

### Phase 1: 프로토타입
1. ✅ xterm.js 기반 채팅 터미널 생성
2. ✅ Mock AI 응답으로 UI 테스트
3. ✅ 색상/스타일링으로 채팅처럼 보이게

### Phase 2: AI 통합
1. Claude Code / Gemini CLI wrapper 구현
2. node-pty로 프로세스 실행
3. 응답 스트리밍 및 파싱

### Phase 3: Context 통합
1. 앱 상태를 context로 수집
2. CLAUDE.md 형식으로 전달
3. 응답을 앱 액션으로 변환

### Phase 4: UI 개선
1. Split view 옵션 추가
2. 터미널 테마 적용
3. Markdown/Code 렌더링

---

## 추가 아이디어

### 1. Context 자동 전달
```typescript
// 사용자가 캔버스 요소 선택 → 자동으로 context에 추가
this.selectionService.onDidChange(() => {
    this.chatTerminal.updateContext({
        selected: this.selectionService.getSelected()
    });
});
```

### 2. 스트리밍 응답
```typescript
this.writeLine(`◆ AI: `);
for await (const chunk of this.aiWrapper.streamResponse(prompt)) {
    this.write(chunk); // 실시간 타이핑 효과
}
```

### 3. 터미널 명령 실행
```typescript
// AI가 제안한 명령을 실제 터미널에서 실행
if (response.includesCommand) {
    this.writeLine(`\x1b[33m[Execute?]\x1b[0m ${response.command}`);
    this.onConfirm(() => {
        this.realTerminal.sendText(response.command);
    });
}
```

---

## 관련 리소스

- **Claude Agent SDK**: `ideas&external_references/claude-agent-sdk/`
- **Gemini CLI**: `ideas&external_references/gemini-cli/`
- **Theia AI Packages**: All `@theia/ai-*` packages in monorepo
- **xterm.js Docs**: https://xtermjs.org/
- **node-pty**: Used by `@theia/terminal`
