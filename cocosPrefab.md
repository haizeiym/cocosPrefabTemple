### pop
#### 弹窗模版
```ts
import { _decorator, Node, SpriteFrame } from "cc";
import { BaseComponent, BindUI, ResLoad } from "lsscript";
const { ccclass } = _decorator;

type callback = (comp?: BaseComponent) => boolean | void;

@ccclass("FileName")
export class FileName extends BaseComponent {
    private _bindUI: BindUI;
    private _sureCall: callback;
    private _cancelCall: callback;
    private _onDesCall: () => void;
    private _imgContent: { bundleName: string; resPath: string } | SpriteFrame;

    public setInit(args: {
        parent?: Node;
        imgContent?: { bundleName: string; resPath: string } | SpriteFrame;
        sureCall?: callback;
        cancelCall?: callback;
        onDesCall?: () => void;
    }): void {
        this._sureCall = args.sureCall;
        this._cancelCall = args.cancelCall;
        this._onDesCall = args.onDesCall;
        this._imgContent = args.imgContent;
        if (args?.parent?.isValid) {
            this._setInit(args.parent);
        } else {
            this.init();
        }
        if (this._imgContent) {
            if ("bundleName" in this._imgContent) {
                ResLoad.spriteFrame(this._imgContent.bundleName, this._imgContent.resPath, true).then((spriteFrame) => {
                    if (!this?.isValid) {
                        this.NodeDestroy();
                        return;
                    }
                    this._bindUI.Img("ImgContent").spriteFrame = spriteFrame;
                });
            } else {
                this._bindUI.Img("ImgContent").spriteFrame = this._imgContent as SpriteFrame;
            }
        }
    }

    protected _initView(): void {
        this._bindUI = this._getUI(this.node);
    }

    protected _initEvent(): void {
        this._addClick(this._bindUI.Btn("BtnClose"), this.NodeDestroy);

        this._addClick(this._bindUI.Btn("BtnCancle"), () => {
            if (typeof this._cancelCall === "function") {
                const res = this._cancelCall(this);
                if (!res) this.NodeDestroy();
                return;
            }
            this.NodeDestroy();
        });

        this._addClick(this._bindUI.Btn("BtnSure"), () => {
            if (typeof this._sureCall === "function") {
                const res = this._sureCall(this);
                if (!res) this.NodeDestroy();
                return;
            }
            this.NodeDestroy();
        });
    }

    protected onDestroy(): void {
        this?._onDesCall();
    }

    protected _destroyBefore(): void {}
}

```

### moreClickChose
#### 多个按钮选择模板，每个item 都要普通背景，选中背景，及展示图片  结构参考moreClickChose.json
```ts
import { _decorator, instantiate, Node, SpriteFrame } from "cc";
import { BaseComponent, BindUI, ResLoad } from "lsscript";
import { VH } from "../common/VHLayout";
const { ccclass } = _decorator;

@ccclass("FileName")
export class FileName extends BaseComponent {
    private _bindUI: BindUI;
    private _onDesCall: () => void;

    private _nodeItem: Node;
    private _nodeContent: Node;

    private _bindUIs: BindUI[];
    private _lastBindUI: BindUI;

    private _clickItemCall: (bindUI?: BindUI) => void;

    public async setInit(args: {
        bundleName: string;
        resPath: string;
        parent?: Node;
        defaultIndex?: number;
        arrangeType?: 0 | 1 | 2; //0为grid，1为TopToBottom，2为LeftToRight
        space?: number;
        spaceX?: number;
        spaceY?: number;
        gridXNum?: number;
        filterStr?: string;
        clickItemCall?: (bindUI?: BindUI) => void;
        onDesCall?: () => void;
    }): Promise<void> {
        this._clickItemCall = args.clickItemCall;
        this._onDesCall = args.onDesCall;
        if (args?.parent?.isValid) {
            this._setInit(args.parent);
        } else {
            this.init();
        }
        let imgs = await ResLoad.dirT(args.bundleName, args.resPath, SpriteFrame, true);
        if (!this?.isValid) return this.NodeDestroy();
        if (args.filterStr) {
            imgs = imgs.filter((e) => e.name.includes(args.filterStr));
        }
        const collator = new Intl.Collator(undefined, { numeric: true });
        imgs.sort((a, b) => collator.compare(a.name, b.name));

        args.defaultIndex = Math.min(args.defaultIndex || 0, imgs.length - 1);
        const bindUI = this._getUI(this._nodeItem);
        this._show(bindUI, false);
        this._bindUIs = [bindUI];
        for (let i = 0, l = imgs.length - 1; i < l; i++) {
            const element = instantiate(this._nodeItem);
            element.setParent(this._nodeContent);
            this._bindUIs.push(this._getUI(element));
        }

        this._bindUIs.forEach((bindUI, index) => {
            bindUI.Img("ImgContent").spriteFrame = imgs[index];
            bindUI.Data = index;
            this._addClick(bindUI.BNode, this._itemClick.bind(this, bindUI));
        });

        const nodes = this._bindUIs.map((bindUI) => bindUI.BNode);
        const arrangeType = args.arrangeType ?? 0;
        if (arrangeType === 1) {
            VH.setVerLayout(this._nodeContent, nodes, args.space ?? 10);
        } else if (arrangeType === 2) {
            VH.setHorLayout(this._nodeContent, nodes, args.space ?? 10);
        } else {
            VH.setGridLayout(
                this._nodeContent,
                nodes,
                args.gridXNum || 4,
                args.spaceX ?? args.space ?? 10,
                args.spaceY ?? args.space ?? 10
            );
        }

        this._lastBindUI = this._bindUIs[args.defaultIndex];
        this._show(this._lastBindUI, true);
        this._clickItemCall?.(this._lastBindUI);
        this._bindUIs.forEach((bindUI) => {
            this._UIO(bindUI.BNode).opacity = 255;
        });
    }

    protected _initView(): void {
        this._bindUI = this._getUI(this.node);
        this._nodeContent = this._bindUI.NodeOnce("NodeContent") || this._bindUI.BNode;
        this._nodeItem = this._bindUI.Node("NodeItem");
    }

    private _itemClick(bindUI: BindUI) {
        if (this._lastBindUI === bindUI) return;
        this._show(this._lastBindUI, false);
        this._show(bindUI, true);
        this._lastBindUI = bindUI;
        this._clickItemCall?.(bindUI);
    }

    private _show(bindUI: BindUI, isShowSelect: boolean = false) {
        this._UIO(bindUI.Node("NodeNormal")).opacity = isShowSelect ? 0 : 255;
        this._UIO(bindUI.Node("NodeSelected")).opacity = isShowSelect ? 255 : 0;
    }

    protected _initEvent(): void {
        if (this._bindUI.Btn("BtnClose")) {
            this._addClick(this._bindUI.Btn("BtnClose"), this.NodeDestroy);
        }
    }

    protected onDestroy(): void {
        this._onDesCall?.();
    }

    protected _destroyBefore(): void {}
}



```

### LobbyHead
#### 个人信息显示模板包括头像(头像框，头像)，名称(背景，名称)，金币(背景，金币数量) 结构参考LobbyHead.json
```ts
import { _decorator, Label, Node, Sprite } from "cc";
import { BaseComponent, BindUI } from "lsscript";
const { ccclass } = _decorator;

@ccclass("FileName")
export class FileName extends BaseComponent {
    private _bindUI: BindUI;

    private _btnHeadCall: ((comp?: BaseComponent) => void) | null = null;

    public setInit(args: {
        parent?: Node;
        compGetCall?: (comps: { bindUI?: BindUI; imgHead?: Sprite; txtName?: Label; txtMoney?: Label }) => void;
        onHeadCall?: (comp?: BaseComponent) => void;
    }): void {
        if (args.parent?.isValid) {
            this._setInit(args.parent);
        } else {
            this.init();
        }

        args.compGetCall?.({
            bindUI: this._bindUI,
            imgHead: this._bindUI.Img("ImgHead"),
            txtName: this._bindUI.Txt("TxtName"),
            txtMoney: this._bindUI.Txt("TxtMoney")
        });
        this._btnHeadCall = args.onHeadCall;
    }

    protected _initView(): void {
        this._bindUI = this._getUI(this.node);
        this._bindUI.NodeOnce("NodeHead");
    }

    protected _initEvent(): void {
        this._addClick(this._bindUI.Node("NodeHead"), () => {
            this._btnHeadCall?.(this);
        });
    }

    protected _destroyBefore(): void {}
}

```

### VoiceSet
#### 声音设置类型  0为所有音效，1为音效，2为背景音乐 结构参考VoiceSet.json
```ts
import { _decorator, CCInteger, Node } from "cc";
import { BaseComponent, BindUI, GG } from "lsscript";
const { ccclass, property } = _decorator;

@ccclass("FileName")
export class FileName extends BaseComponent {
    @property({ displayName: "声音设置类型 0为所有音效，1为音效，2为背景音乐" })
    private _setType: 0 | 1 | 2 = 0;
    @property({
        type: CCInteger,
        displayName: "声音设置类型 0为所有音效，1为音效，2为背景音乐"
    })
    public set setType(value: 0 | 1 | 2) {
        this._setType = value;
    }

    public get setType() {
        return this._setType;
    }

    private _bindUI: BindUI;

    public setInit(args?: { parent?: Node }): void {
        if (args?.parent?.isValid) {
            this._setInit(args.parent);
        } else {
            this.init();
        }
        GG.clsExtra.isInitAudio();
        if (this._setType === 0) {
            this._setStatus(GG.clsExtra.getIsStopAudio());
        } else if (this._setType === 1) {
            this._setStatus(GG.clsExtra.getIsStopEffect());
        } else if (this._setType === 2) {
            this._setStatus(GG.clsExtra.getIsStopBgm());
        }
    }

    protected _initView(): void {
        this._bindUI = this._getUI(this.node);
    }

    protected _initEvent(): void {
        this._addClick(this.node, this._voiceSetCall);
    }

    private _voiceSetCall() {
        if (this._setType === 0) {
            GG.clsExtra.stopAudio(!GG.clsExtra.getIsStopAudio());
            this._setStatus(GG.clsExtra.getIsStopAudio());
        } else if (this._setType === 1) {
            GG.clsExtra.stopEffect(!GG.clsExtra.getIsStopEffect());
            this._setStatus(GG.clsExtra.getIsStopEffect());
        } else if (this._setType === 2) {
            GG.clsExtra.stopBgm(!GG.clsExtra.getIsStopBgm());
            this._setStatus(GG.clsExtra.getIsStopBgm());
        }
    }

    private _setStatus(isOpen: boolean) {
        const nodeOpen = this._bindUI.Node("NodeOpen");
        nodeOpen.x = isOpen ? 0 : -1000000;
        this._UIO(nodeOpen).opacity = isOpen ? 255 : 0;

        const nodeClose = this._bindUI.Node("NodeClose");
        this._UIO(nodeClose).opacity = isOpen ? 0 : 255;
        nodeClose.x = isOpen ? -1000000 : 0;
    }

    protected _destroyBefore(): void {}
}

```