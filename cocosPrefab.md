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

```