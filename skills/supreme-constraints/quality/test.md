# 集测质量约束

## 一、编写形式规范

### 1.1 必须遵守
1. **所有集测用例必须编写代码**，禁止使用纯文档形式或手动测试步骤
2. **所有集测 mock 必须按集测框架的要求写成代码**（例如 pytest fixture）

### 1.2 绝对禁止
- `python -c "..."` 或 `python <<< "..."` 或 heredoc 形式: `python << 'EOF' ... EOF`
- 任何其他形式的内联脚本

### 1.3 目录结构
```
tests/integration/
├── __init__.py
├── conftest.py           # pytest 配置和 fixtures
├── test_xxx.py           # 具体测试模块
└── utils/                # 测试工具
```

### 1.4 文件位置
- ⚠️**集测代码是主要产出**，必须保存到集测代码目录
- 集测代码必须规范地写入集测代码目录合适位置
- 辅助代码（工具类）必须规范地按集测框架要求写入集测代码目录合适位置
  - 例如，所有公共 fixture 都在 `conftest.py` 中定义

### 1.5 正确示例
```python
# tests/integration/utils/mock_http_server.py
class MockHttpServer:
    """Mock HTTP服务器，用于测试代理转发功能。"""
    def __init__(self, port: int, response: str):
        self.port = port
        self.response = response
        self._process: Optional[subprocess.Popen] = None

    def start(self) -> None:
        """启动服务器。"""
        ...

    def stop(self) -> None:
        """停止服务器。"""
        ...

# tests/integration/conftest.py
@pytest.fixture
def mock_http_server() -> Generator[MockHttpServer, None, None]:
    """提供Mock HTTP服务器fixture。"""
    server = MockHttpServer(port=8080, response="OK")
    server.start()
    yield server
    server.stop()
```

## 二、代码规范

### 2.1 代码风格
- **所有 Python 代码必须有完整的 Type Annotation**
- 命名清晰（集测函数名应描述集测场景）

### 2.2 资源管理
- 使用 fixtures 管理集测环境
- 资源管理正确（启动/停止、创建/销毁）
- 正确使用 context manager 管理资源
- 有适当的等待/超时机制
- 端口/资源正确清理
- 进程正确终止

### 2.3 文件与环境管理
- 检查被测的可执行文件存在与否并正确提示用户
- 规范管理集测产出物，不污染无关目录
- 规范管理集测配置文件，不污染正常配置文件目录

## 三、可重复性规范
- 集测可独立运行（无集测间依赖）
- 集测可重复运行（无状态残留）

## 四、黑盒测试原则

### 4.1 测试方式
- 只通过外部接口测试（不依赖内部代码）
- 避免调用被测系统内部代码

### 4.2 测试工具
- 使用真实工具（curl、nc 等）
- 根据场景选择正确的集测工具（curl/socket等）

## 五、Mock/Fixture 设计规范
- Mock 设计必须合理
- Mock 响应必须符合协议规范
- Mock 必须有全面的错误模拟能力
- Fixture 生命周期必须正确（setup/teardown）

## 六、覆盖率要求

### 6.1 覆盖范围
- 覆盖架构文档所有功能点
- 覆盖所有用户场景
- 覆盖所有接口端点
- 覆盖正常场景
- 覆盖异常场景
- 覆盖边界场景

### 6.2 覆盖率追踪
- 覆盖率追踪表必须完整
- 每个功能点必须有对应集测用例
