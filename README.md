<?php

namespace ResultSystems\PdoCrud;

class Crud
{
    protected $drive = 'pgsql';
    protected $cn    = null,
    $db              = null,
    $host            = null,
    $username        = null,
    $password        = null,
    $table           = null;

    /**
     * Classe construtora
     *
     * @param string $db
     * @param string $host
     * @param string $username
     * @param string $password
     * @param string $table
     */
    public function __construct($db = null, $host = null, $username = null, $password = null, $table = null)
    {
        if (!is_null($db)) {
            $this->db = $db;
        }

        if (!is_null($host)) {
            $this->host = $host;
        }

        if (!is_null($username)) {
            $this->username = $username;
        }

        if (!is_null($password)) {
            $this->password = $password;
        }

        if (!is_null($table)) {
            $this->table = $table;
        }
    }

    /**
     * set Table
     *
     * @param string $table
     */
    public function setTable($table)
    {
        $this->table = $table;

        return $this;
    }

    /**
     * set Connect
     *
     * @param  string $db
     * @param  string $host
     * @param  string $username
     * @param  string $password
     */
    public function connect($db = null, $host = null, $username = null, $password = null)
    {
        $this->cn = new PDO($this->drive . ':dbname=${db};host=${host}', $username, $password);
        $this->cn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

        return $this;
    }

    /**
     * cria item
     *
     * @param string $data
     * @param string $table
     */
    public function create($data, $table = null)
    {
        if (is_null($this->cn)) {
            return;
        }

        $this->setTable($table);

        $sql        = "INSERT INTO " . $this->table . " VALUES ";
        $values     = '';
        $parameters = [];
        foreach ($data as $key => $value) {
            if ($values != '') {
                $values .= ", ";
            }

            $values .= $key . " = :" . $key;
            $parameters[] = [':' . $key => $value];
        }
        $sql .= "(" . $values . ")";

        try {
            $stmt = $this->cn->prepare($sql);

            $stmt->execute($parameters);

            return $stmt->rowCount();
        } catch (PDOException $e) {
            return $e->getMessage();
        }
    }

    /**
     * atualiza registro
     *
     * @param  array  $data
     * @param  int $id
     * @param  string $column
     * @param  string $table
     */
    public function update(array $data, $id, $column = 'id', $table = null)
    {
        if (is_null($this->cn)) {
            return;
        }

        $this->setTable($table);

        $sql        = "UPDATE " . $this->table . " SET ";
        $sets       = '';
        $parameters = [];
        foreach ($data as $key => $value) {
            if ($sets != '') {
                $sets .= ", ";
            }

            $sets .= $key . " = :" . $key;
            $parameters[] = [':' . $key => $value];
        }
        $sql .= $sets;

        $sql .= " ${column} = ${id} ";

        try {
            $stmt = $this->cn->prepare($sql);
            $stmt->execute($parameters);

            return $stmt->rowCount();
        } catch (PDOException $e) {
            return $e->getMessage();
        }
    }

    /**
     * apaga item
     *
     * @param  int $id
     * @param  string $column
     * @param  string $table
     */
    public function delete($id, $column = 'id', $table = null)
    {
        if (is_null($this->cn)) {
            return;
        }

        $this->setTable($table);

        try {
            $stmt = $this->cn->prepare("DELETE FROM " . $this->table . " WHERE ${column} = :id");
            $stmt->bindParam(':id', $id);
            $stmt->execute();

            return $stmt->rowCount();
        } catch (PDOException $e) {
            return $e->getMessage();
        }
    }

    /**
     * pega item(ns)
     *
     * @param  string $id
     * @param  string $column
     * @param  string $table
     */
    public function get($id = null, $column = 'id', $table = null)
    {
        if (is_null($this->cn)) {
            return;
        }

        $this->setTable($table);

        $sql = "SELECT * FROM " . $this->table;

        if (!is_null($id)) {
            $sql .= " WHERE ${column} = ${id}";
        }

        $consulta = $this->cn->query($sql);

        try {
            return $consulta->fetchAll(PDO::FETCH_ASSOC);
        } catch (PDOException $e) {
            return $e->getMessage();
        }
    }
}
