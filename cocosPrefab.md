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
#### 多个按钮选择模板，每个item 都要普通背景，选中背景，及展示图片
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
        gridXNum?: number;
        spaceX?: number;
        spaceY?: number;
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
        const imgs = await ResLoad.dirT(args.bundleName, args.resPath, SpriteFrame, true);
        if (!this?.isValid) return this.NodeDestroy();

        args.gridXNum = args.gridXNum || 4;
        args.defaultIndex = args.defaultIndex || 0;
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

        VH.setGridLayout(
            this._nodeContent,
            this._bindUIs.map((bindUI) => bindUI.BNode),
            args.gridXNum,
            args.spaceX ?? 10,
            args.spaceY ?? 10,
            true
        );

        this._lastBindUI = this._bindUIs[args.defaultIndex];
        this._show(this._lastBindUI, true);
        this._itemClick(this._lastBindUI);
        this._bindUIs.forEach((bindUI) => {
            this._UIO(bindUI.BNode).opacity = 255;
        });
    }

    protected _initView(): void {
        this._bindUI = this._getUI(this.node);
        this._nodeContent = this._bindUI.NodeOnce("NodeContent");
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
        this._addClick(this._bindUI.Btn("BtnClose"), this.NodeDestroy);
    }

    protected onDestroy(): void {
        this?._onDesCall();
    }

    protected _destroyBefore(): void {}
}



```