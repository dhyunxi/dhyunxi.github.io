



# autograd
---
- torch.autograd.backward( tensors , retain_graph , create_graph , grad_tensors)
	- 反向传播
	- tensor的类方法backward实际就是调用此函数
	- gradient=grad_tensors在loss不止1维时调整各项权重
- torch.autograd.grad(outputs , inputs , create_graph , retain_graph , grad_tensors)
	- 求取梯度
	- create_graph=True创建计算图，多用于高阶导
		- grad1=torch.autograd.grad(y,x,create_grad=True)：y对x求导，允许grad1创建计算图
		- grad2=torch.autograd.grad( grad_1[0] , x )：二阶导
## 注意点
- 梯度不自动清零
- 依赖于叶子结点的结点默认require_grad=True
- 叶子结点不能in-place操作
	- 非in-place操作会更改地址，造成属性变化，如赋值
