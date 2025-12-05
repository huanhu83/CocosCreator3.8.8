# CocosCreator3.8.8
自定义引擎修改一些BUG
1.输入时字符不能超过限定长度
    修改文件：engine\platforms\minigame\common\engine\Editbox.js
    _registerKeyboardEvent () {
        ...

        cbs.onKeyboardInput = function (res) {
            //#region [自定义]，添加如下代码，输入内容超出限定长度后截断
            if(res.value.length>delegate.maxLength){
                res.value = res.value.substr(0,delegate.maxLength);
            }
            //#endregion

            if (delegate._string !== res.value) {
                delegate._editBoxTextChanged(res.value);
            }
        };

        ...
    }

     _showKeyboard () {
        const delegate = this._delegate;
        const multiline = (delegate.inputMode === EditBoxComp.InputMode.ANY);
        __globalAdapter.showKeyboard({
            defaultValue: delegate.string,
            maxLength: MAX_VALUE, //delegate.maxLength < 0 ? MAX_VALUE : delegate.maxLength, [自定义]
            multiple: multiline,
            confirmHold: false,
            confirmType: getKeyboardReturnType(delegate.returnType),
            success (res) {

            },
            fail (res) {
                cc.warn(res.errMsg);
            },
        });
    },

2.底层渲染报错的保护
    修改文件：engine\cocos\2d\components\graphics.ts
    protected _render(render: IBatcher): void {
        //#region [自定义]，保护渲染报错
        if (!this.getMaterialInstance(0)) {
            return;
        }
        //endregion

        if (this._isNeedUploadData) {
            if (this.impl) {
                const renderDataList = this.impl.getRenderDataList();
                const len = this.model!.subModels.length;
                if (renderDataList.length > len) {
                    for (let i = len; i < renderDataList.length; i++) {
                        this.activeSubModel(i);
                    }
                }
            }
            this._uploadData();
        }

        render.commitModel(this, this.model, this.getMaterialInstance(0));
    }

3~7 仅小游戏有问题，Native项目就不同步了（后续需要参考其他几个版本的自定义修改）

8.修复平铺贴图有接缝的BUG(3.8.7)
    修改文件：engine\cocos\2d\assembler\tiled.ts

    注释掉updateRenderData方法中的一句代码：
    // dynamicAtlasManager.packToDynamicAtlas(sprite, frame); //[自定义]，解决平铺贴图有接缝的问题

