 terminbot/db/ 目录下目前主要用 SQLAlchemy（见 db/utils.py 和 db/models.py）实现了用户与预约的配置和操作，暂未集成 Supabase 日志逻辑。
 
 下面结合你现有结构，给出日志表、日志存储函数和 DB 类重构的建议与代码实现。

1. 创建日志表（models.py）
在 db/models.py 新增 Log（或 SupabaseLog）模型：

from sqlalchemy import JSON

class Log(Base):
    __tablename__ = 'logs'
    id = Column(BigInteger, primary_key=True)
    timestamp = Column(DateTime, default=func.now())
    level = Column(String(20), nullable=False)   # 'DEBUG', 'INFO', 'WARNING', 'ERROR'
    message = Column(Text, nullable=False)
    metadata = Column(JSON)   # 结构化字段，便于存更多信息
    
2. 日志存储函数
在 db/utils.py 新增日志写入函数：

from db.models import Log

def save_log(level: str, message: str, metadata: dict = None):
    """将日志写入数据库"""
    session = SessionLocal()
    try:
        log = Log(level=level, message=message, metadata=metadata or {})
        session.add(log)
        session.commit()
    except Exception as e:
        print(f"日志写入失败: {e}")
        session.rollback()
    finally:
        session.close()
3. DB 类重构建议
为了统一管理数据库操作，建议封装一个 DB 类，将常用的增删查改及日志相关操作都放进来，示例：

from db.utils import SessionLocal
from db.models import AppointmentProfile, Log
from typing import List, Optional

class DB:
    def __init__(self):
        self.Session = SessionLocal

    def save_log(self, level: str, message: str, metadata: dict = None):
        session = self.Session()
        try:
            log = Log(level=level, message=message, metadata=metadata or {})
            session.add(log)
            session.commit()
        except Exception as e:
            print(f"日志写入失败: {e}")
            session.rollback()
        finally:
            session.close()
    
    def get_all_appointment_profiles(self) -> List[AppointmentProfile]:
        session = self.Session()
        try:
            profiles = session.query(AppointmentProfile).all()
            return profiles
        finally:
            session.close()
    
    # 可扩展更多业务相关方法
4. 实际用法
新增日志时只需：db.save_log('INFO', '预约成功', {'user_id': 123})
统一通过 DB 实例管理所有数据库和日志相关操作。
5. 迁移/兼容 Supabase
如果你有单独的 Supabase 远程库，也可以在 DB 类中增加 Supabase 客户端的支持，或者通过 SQLAlchemy 连接 Supabase 的 Postgres。
metadata 字段建议用 JSON 类型，Postgres/Supabase 都支持。
6. 总结
db/models.py 中新增 Log 模型
db/utils.py 增加 save_log 函数
新建 db/db.py，把所有操作（包括日志）封装进 DB 类，便于扩展和维护
如需完整代码文件结构示例或 SQLAlchemy → Supabase 兼容建议，可继续向我提问！
（注：搜索结果有限，更多 db/ 目录下内容请到 GitHub 查看：db/ 代码文件列表）